
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
| Build outcomes per edition, toolchain prerequisites, database components | **10** | Records that the build debt exists, with severity and owner; does not restate the diagnosis. The build status of the migration source is 10's to state — it carries MVC 5 as **blocked pending a Windows verification run** ([10 §5.4](10-build-and-deployment-requirements.md)) — and no entry below treats that build as verified |
| The effort model — units, bands, assumptions, confidence | **07** | Supplies the quantities and states which are estimation-safe (section 12) |
| Target framework, SDK band, project-format conversion, per-package outcomes | **04** | Records the debt that makes the conversion harder; names no target version |
| Hosting target, deployment model, path-casing audit, key-ring location | **06** | Records the casing evidence it found by accident (section 10.7) and routes it |
| Security posture, policy, PII and disclosure analysis | **09** | Records the credential, the anti-forgery gap and the exception disclosure as debt entries and routes the analysis |
| The forward anti-forgery policy, the `AddToCart` verb change, DI design, Identity migration, the duplicate-`Genre` browse contract | **05** | Records today's coverage and the measured source behaviour; proposes no policy and authors no competing contract |
| No-successor constructs and differing-default successors | **12** | Records the debt; does not classify migration blockers |
| Workstream sequencing and gates | **03** | Names the owner of each remediation; sequences nothing |

Two further exclusions. This register does not rank debt by remediation cost — that is an effort judgement and belongs to 07. And it does not propose an order of repair: severity is a statement about consequence, not a queue.

### 1.3 The no-modification constraint — and why this deliverable is the one most tempted to break it

The user directed *"Do not make code changes initially"*, and the attached environment setup instructions independently restate the same gate: *"Do not modify code until assessment and modernization plan are approved."* Two inputs agreeing on it is why the boundary extends even to the defects catalogued below.

This is the deliverable that finds the fixable things. Every item in it stays exactly as it is: the stale fourth solution file, the duplicated schema scripts, the fourteen committed database binaries, the two committed `packages/` trees, the three IDE user-state files, the plaintext administrator credential, the unprotected state-changing POSTs. Each is documented; none is repaired.

**The evidence that the constraint held is four commands, and no three of them are the check.** They differ in kind, which is why one cannot stand in for another. One is a statement about a **range** — durable, re-runnable by anyone, and the only one of the four that says anything about history. Two are statements about **one checkout at one moment**, and the second of those is the one that sees generated payload, because a bare porcelain status is blind to ignored paths. The fourth asks what an ignore-aware clean would still remove. Run them against the committed checkpoint:

```bash
# 1 — the tracked diff against the pre-assessment baseline, in four assertions
git diff --name-status ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD
# -> 13 rows, every one an 'A' for a file under docs/modernization/.
#    No 'M' and no 'D' against any file that existed before.
git diff --name-status ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD | wc -l            # -> 13
git diff --name-status ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD | grep -c '^A'     # -> 13
git diff --name-status ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD | grep -vc '^A'    # -> 0
git diff --name-only   ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD \
  | grep -v '^docs/modernization/' | wc -l                                               # -> 0

# 2 — tracked working-tree state
git status --porcelain             # -> (no output)

# 3 — the same state with ignored content included: the one check the other three are blind to
git status --porcelain --ignored   # -> (no output) once ignored output is cleared

# 4 — what an ignore-aware clean would remove, listed rather than deleted
git clean -ndX                     # -> (no output) once cleared; lists any tree present
```

**Only the start of the range is a literal hash, and that asymmetry is deliberate rather than an omission.** `ea2552d6eda7c20e9477a512e5c615665618cf35` is pinned and verifiable — `Merge pull request #9`, the last commit before this engagement — so `ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD` is exactly this engagement and nothing else. The far end is `HEAD` because **no document can contain the hash of the commit that adds it**; a reader who wants a range pinned at both ends resolves this branch's tip once on the checkout in front of them — `git rev-parse HEAD` — and substitutes that value for `HEAD`, which is a one-token substitution the reader can perform and the author cannot. Command 1's four assertions hold at every tip at which the set is complete, including after the commit that lands the last correction to it, because each is a property of the range rather than of a particular tip. Together the four say the same thing from four directions: everything the engagement committed is an addition under `docs/modernization/`, and **no pre-existing repository file was modified or deleted**. During authoring, command 2 instead listed these thirteen files as untracked, which is the same fact at an earlier moment — and commands 3 and 4 are recorded here because, for as long as the assessment's own restore and build output sat in the checkout, commands 1 and 2 both reported a clean tree while generated payload was present. [Deliverable 02 §1.3](02-dependency-inventory.md) owns the per-tree record of that output and its removal; this section adopts the same four-command check and does not restate the counts.

All verification for this document was read-only. The one experiment that needed to write anything — the `core.ignorecase` probe of section 10.7 — was run in a throwaway repository created outside the checkout and deleted afterwards, and is flagged where it appears.

### 1.4 Authoring contract, and the absence of user rules

**No user-specified rules were provided for this project.** `review_rules` returns exactly *"No user rules provided."*, re-verified while authoring this document. There is consequently no rule to name, summarize or cite, and no file forced into scope by one. The absence is not licence to lower the bar: this register is held to enterprise-standard best practice, expressed as the four contracts below.

- **Citation.** Every claim about the existing system carries an inline `[<path>:<locator>]` citation at the point of the claim, with a repository-relative path that resolves in the checkout. For a text artifact the locator is a line or a line range. For evidence that has no line to point at — a committed database binary, the tutorial PDF, the restore executable — the locator is the property that does resolve: a byte size, together with the command that reads it, never a line number the artifact cannot support. There is no trailing reference list, because a citation collected at the end cannot be checked against the sentence it supports.
- **Reproducibility.** A claim that ranges over the repository — a count, a total, an absence — has no single line to point at, so its evidence is the command that produces it, stated adjacent to the claim and collected once more in section 14. Byte totals are summed with `awk`; `bc` is not installed on the verification host.
- **Primacy of repository evidence.** Where this document and any other input disagree, the repository governs. Technical Specification §3.3 is cited only as a secondary cross-reference alongside repository evidence, never instead of it. Two places where a stated figure did not reproduce exactly are recorded openly, in sections 3.4 and 10.7, with the measured value and the command that produces it.
- **One fact, one owner.** Section 1.2 is the boundary; section 13 is the routing map.

Commands are given in canonical POSIX form. On the Windows verification host they were run through the bundled Git-for-Windows `bash` from the repository root, and every value shown after `# ->` is the value observed there.

Markdown line length follows the corpus convention recorded in [`README.md` § 10 Markdown line-length convention](README.md#10-markdown-line-length-convention).

### 1.5 How to read an entry

Each entry has an identifier (`F-08-nn`), a severity, the evidence, a remediation and an owner.

**One remediation per entry, and it states one action, one owner and — where the entry carries one — one acceptance condition.** This is an invariant of the register rather than a stylistic preference, because two remediations on one entry are two instructions to whoever executes it, and an implementer who follows the wrong one has still followed the register. So the remediation appears exactly once, after the evidence it follows from; where the action has an unavoidable decision point — F-08-11's history step, which only the repository owner can take — it is stated as **one action with a named choice inside it** rather than as competing options, and the entry says so in those words. The command in section 14 proves the invariant mechanically: exactly 28 `Remediation` rows and 28 `Owner` rows for 28 entries.

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

> **382 is the figure deliverable 07 estimates against** — non-blank lines, the sizing metric over the whole file [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:1-423]. **The 422 and 426 quoted in section 3.3 are physical line counts and belong to the duplication comparison only.** They are not sizing figures and must not be used as one. The citation bound above is `423` rather than `422` for a reason worth stating once, since this section owns the counting rules: the file has no terminal newline, so it carries 423 content lines while `wc -l` — which counts newline terminators — reports 422. The locator has to span all 423 or it would exclude the closing brace, which the 382 non-blank count includes.

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

Eight are byte-identical; `Album.cs` shows **2 diff lines**. Both editions' `ViewModels` folders hold the same two files, `ShoppingCartViewModel.cs` and `ShoppingCartRemoveViewModel.cs`, and both pairs are byte-identical. The seed — **826 LF, which is 827 content lines**, the file having no terminal newline [src/MVC5/MvcMusicStore/Models/SampleData.cs:1-827] — is among the byte-identical files, so the largest single file in each edition is a verbatim copy of the other. Deliverable 01 §2.4 owns the metric definition and gives the reproducing commands; the figure here is the physical-line metric used for duplication, never the non-blank sizing metric.

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

**First, MVC 3's cart owns its own context and commits internally.** `ShoppingCart` declares its own instance — `MusicStoreEntities storeDB = new MusicStoreEntities();` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:11] — and calls `SaveChanges()` at **five** points inside the cart itself [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:57], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:82], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:98], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:156], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:197]. Its `GetCart` overloads accordingly take **no context parameter** [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:17], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:25]. MVC 5's cart is the opposite design: it contains **zero** `SaveChanges()` calls, and both its overloads require the caller's context [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:21], [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:29], so the caller owns the unit of work.

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
| **Remediation** | Do **not** remediate by deleting or merging editions. Retain MVC 4 and MVC 3 read-only as **comparative and historical references** — MVC 5 is the sole executable behavioural baseline the port is validated against, and neither MVC 4 nor MVC 3 is driven by the characterization suite that captures it (F-08-15; architecture in [05](05-aspnet-core-migration-approach.md)) — and let the duplication end naturally when one target application replaces the migration source. Deliverable 07 must size MVC 3 from its own measurements, never by analogy with MVC 4 or MVC 5. |
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
| **Editions** | **The finding is the duplication, and the duplication is MVC 5 only.** The registration itself is **common to all three editions** — MVC 4 and MVC 3 each perform it exactly once, so neither has this defect and neither is missing the initializer. |

`Database.SetInitializer(new SampleData())` is called **twice** during MVC 5's startup: once from the ASP.NET application object [src/MVC5/MvcMusicStore/Global.asax.cs:20] and once from the OWIN startup partial [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:16] — the two entry points deliverable 01 §3.1 records.

**What the other two editions do, stated because the difference is one of count, not of kind.** Both register the same initializer with the same seed class, once each:

| Edition | Registrations of `SampleData` | Where | Why one and not two |
| --- | --- | --- | --- |
| **MVC 5** | **2** | [src/MVC5/MvcMusicStore/Global.asax.cs:20] and [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:16] | It has two composition roots and both claim database bootstrapping |
| **MVC 4** | **1** | [src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:16], inside `AppConfig.Configure()`, which `Global.asax.cs` calls | One composition root: the application object delegates to `AppConfig` rather than registering anything itself |
| **MVC 3** | **1** | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Global.asax.cs:34], directly in `Application_Start` | It has no `App_Start` folder, so there is nowhere else for a second registration to live |

`git grep -n 'SetInitializer' -- 'src/'` returns **five** lines outside the committed package payloads, and the fifth must not be miscounted as a sixth `SampleData` registration: `Database.SetInitializer<UsersContext>(null)` [src/MVC4/MvcMusicStore/Filters/InitializeSimpleMembershipAttribute.cs:28] does the opposite — it **disables** initialization for MVC 4's SimpleMembership context. So the correct census is four `SampleData` registrations across the three editions, distributed 2 / 1 / 1, plus one deliberate suppression in MVC 4.

**What this is not.** `SetInitializer<TContext>` **sets** the strategy for a context type; it does not add to a list. The second call replaces the first, and exactly one initialization strategy is in force at runtime. This is **duplicated startup configuration, not a doubled destructive path** — the destructive behaviour is F-08-10's, and it would occur exactly once either way. Overstating this entry as "the database is initialized twice" would misrepresent the runtime.

**Why it is nonetheless worth an entry.** Two files each believe they own database bootstrapping, and the repository's own documentation records the duplication as though it were intentional: the seed class is described as "configured as the database initializer in `Global.asax.cs` and `App_Start/Startup.App.cs`" [src/MVC5/README.md:31]. During the port, the two composition roots collapse into one, and a reader who trusts either file alone will conclude something different about who owns schema lifecycle. Deliverable 01 §3.4 records the same fact structurally. Note what the entry is **not** claiming: the destructive initializer is not an MVC 5 peculiarity — it is registered in all three editions and F-08-10 carries it for all three at Critical. What is peculiar to MVC 5 is that two files register it.

| | |
| --- | --- |
| **Remediation** | One registration in one place. Under the target composition the question disappears with the two entry points, so the remediation is to make the ownership explicit while porting rather than to patch the current file. |
| **Owner** | The migration workstream (startup composition) |

### 5.2 F-08-03 — Per-page query fan-out from the shared layout (all three editions, in two different query shapes)

| | |
| --- | --- |
| **F-08-03** | **Two uncached queries execute on every page render, and in two of the three editions one of them is a nested aggregate** |
| **Severity** | **High** — the cost is paid by every request to every page, and in MVC 5 and MVC 4 one of the two queries aggregates across three tables |
| **Editions** | All three carry the layout-level fan-out; **the query shapes are not the same in all three** and the table below states each. MVC 5 renders 2 child actions in the layout, MVC 4 2 of its 4, MVC 3 2 of its 2 — deliverable 01 §5.3 owns the per-edition declaration and call-site counts |

Both child actions are invoked from the shared layout, so they run for every view that uses it — which is every view, since `_ViewStart.cshtml` sets the layout globally [src/MVC5/MvcMusicStore/Views/_ViewStart.cshtml:2]. The call sites differ by edition in helper and in line, and all six are located here rather than generalized from MVC 5's pair: `@Html.Action("GenreMenu", "Store")` [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:25] and `@Html.Action("CartSummary", "ShoppingCart")` [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:26]; the same two helpers in MVC 4 at [src/MVC4/MvcMusicStore/Views/Shared/_Layout.cshtml:32] and [src/MVC4/MvcMusicStore/Views/Shared/_Layout.cshtml:25]; and `Html.RenderAction` rather than `@Html.Action` in MVC 3, at [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Views/Shared/_Layout.cshtml:21] and [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Views/Shared/_Layout.cshtml:16].

**The nested aggregate exists in MVC 5 and MVC 4 only.** MVC 3's `GenreMenu` is a different query, and describing one shape as if it held in all three would misstate both the cost and the remediation:

| Edition | `GenreMenu` query shape | `CartSummary` query shape |
| --- | --- | --- |
| **MVC 5** | **Nested aggregate.** Orders genres by a `Sum` over each genre's albums' order-detail quantities, then `Take(9)` [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:46-52]. The ordering key is derived from the entire order history, and nothing memoizes it | Loads the cart, projects and orders the titles, then enumerates the sequence **twice** — once for `Count()` and once for `Distinct()` [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:89-96] |
| **MVC 4** | **The same nested aggregate** [src/MVC4/MvcMusicStore/Controllers/StoreController.cs:46-52] — the file is byte-identical to MVC 5's per section 3.1, so the shape is identical by measurement rather than by assumption | The same double enumeration [src/MVC4/MvcMusicStore/Controllers/ShoppingCartController.cs:89-96]; `ShoppingCartController.cs` is also byte-identical per section 3.1 |
| **MVC 3** | **No aggregate at all.** `storeDB.Genres.ToList()` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreController.cs:52] — no `Sum`, no ordering and **no `Take`**, so it materializes the whole genre table on every page. Cheaper per row than the aggregate and unbounded in a way the other two are not; deliverable 01 §9.2 row 5 records the same difference, as an unranked menu against MVC 5's ranked top nine | **One count, no title list.** `cart.GetCount()` into `ViewData` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/ShoppingCartController.cs:82-90] — no projection, no ordering and no second enumeration |

```bash
for e in src/MVC5/MvcMusicStore src/MVC4/MvcMusicStore \
         src/MVC3/MvcMusicStore-Completed/MvcMusicStore; do
  printf '%s: ' "$e"; grep -c 'OrderDetails.Sum' "$e/Controllers/StoreController.cs"
done                                    # -> MVC5 1, MVC4 1, MVC3 0
```

**Four distinct writes change the aggregate, not one.** That is the evidence this register contributes; the caching policy built on it is not this register's to set, and section 5.2's remediation row cites its owner rather than stating a second one. The ordering key sums order-detail quantities **grouped by genre**, so it changes on any write that moves quantity between genres or changes which albums a genre owns — which is more than order placement:

| Trigger | Why it changes the aggregate |
| --- | --- |
| **Checkout** | New `OrderDetail` rows add quantity to whichever genres the ordered albums belong to [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:44-51] |
| **Album genre reassignment** | An album edit that changes `GenreId` moves that album's entire accumulated order-detail quantity from one genre's sum to another's, with no order written at all — the `Edit` POST marks the whole entity `Modified` and commits [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:86-94] |
| **Album create** | A new album joins a genre's album set; it contributes zero until ordered, but it enters the set the `Sum` walks [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:53-60] |
| **Album delete** | Removing an album removes its quantity from its genre's sum, which can reorder the top nine [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:116-121] |

#### 5.2.1 What actually invalidates the cached aggregate — four write paths, not one

The genre aggregate is cacheable, but only against a correct statement of what changes it, and the obvious statement is wrong. **Its inputs are not changed only by orders.** Every write below moves the aggregate's value or its input set, and each is reachable from the shipped administration surface:

| Event | Why the aggregate changes | Evidence |
| --- | --- | --- |
| **An order is placed** | New `OrderDetail` rows with quantities enter the `Sum` for whichever genres the ordered albums belong to | `CreateOrder` adds an `OrderDetail` per cart row [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:136-147], committed by the checkout write [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:44-51] |
| **An album edit changes `GenreId`** | **No order is involved.** The POST binds the whole posted `Album` and marks it `EntityState.Modified`, so a changed `GenreId` reclassifies that album's *existing* order-detail quantities out of one genre's `Sum` and into another's — two genres move at once, from a single administrative action | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:86-92], specifically the `EntityState.Modified` assignment at [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:91] and the save at [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:92]; the aggregate it feeds is [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:46-52] |
| **An album is deleted** | The album leaves its genre's album set, and its order-detail quantities leave that genre's `Sum` with it | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:119-121] |
| **An album is created** | The album joins a genre's album set immediately. Its contribution to the `Sum` is zero until it is ordered, so the *ordering* does not move on the create itself — which is precisely why a cache keyed on "orders only" would look correct in testing and drift in production | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:58-59] |

The safe rule follows from the table rather than from case analysis: **invalidate on any committed write to `Albums` or `OrderDetails`**, not on order placement alone. The genre set itself is stable — no controller in any edition writes `Genres`, and in the migration source the seed is the only writer: `genres.ForEach(s => context.Genres.Add(s))` [src/MVC5/MvcMusicStore/Models/SampleData.cs:822], inside `AddGenres` [src/MVC5/MvcMusicStore/Models/SampleData.cs:801-802], reached from `Seed` at [src/MVC5/MvcMusicStore/Models/SampleData.cs:15]. The absence half is a repository-wide claim and carries its command: `git grep -nE 'Genres\.(Add|AddRange|Remove|RemoveRange|Attach)' -- '*.cs'` returns exactly two lines, `src/MVC4/MvcMusicStore/Models/SampleData.cs:822` and `src/MVC5/MvcMusicStore/Models/SampleData.cs:822`, and no controller in any edition appears in the result. So what moves is the ordering and the membership of the top nine, never the existence of a genre.

**Cross-replica propagation is a stamp rotation for three of the four triggers and an expiry for the fourth, and that split is the whole of this entry's consequence.** Deliverable [05](05-aspnet-core-migration-approach.md) §8.2 owns the caching **mechanism** and has taken the decision; deliverable [06](06-azure-hosting-recommendations.md) §6.4 owns only the **sizing** consequence of where it lands. The store is the **SQL-backed distributed cache**, the same `dbo.SessionCache` table session uses, under a **versioned key** — the value at `genre-menu:{stamp}` and the stamp itself at `genre-menu:stamp` — so invalidation is a **rotation of the stamp rather than an eviction**. This register does not re-decide any of that, and records only its direct consequence for invalidation, which the versioned key changes in this entry's favour: a rotation is one small write to a store **every replica reads**, so the three administration writes are reflected on the next page render **everywhere**, not after a delay. What the expiry still covers is the fourth trigger. A checkout moves the ranking and deliberately does **not** rotate the stamp, so its effect converges through the entry's own lifetime — **absolute, at 60 seconds**, absolute rather than sliding for the reason that matters here: a sliding window on a value read during every page render would be refreshed by the reads themselves, so a continuously served instance could hold a stale ordering indefinitely. With an absolute window, a ranking that an order has changed is current within 60 seconds on every replica. That is acceptable for a popularity-ordered navigation menu and would not be for anything transactional, which is the distinction a reader has to be able to make. If a tighter bound on the order trigger is ever required, rotating on the checkout write is the change, and it is 05 §8.2's to make rather than this entry's.

#### 5.2.2 The rest of the contract, cited to its owners — an invalidation rule alone does not make a cache correct

The four write paths above are **this register's** contribution, and the only part of the caching contract it decides. The remaining elements are decisions taken elsewhere, and they are collected here for one reason: an entry that named the invalidation paths and nothing else would be satisfied by an implementation that caches nothing at all, or that caches per user, or that stampedes on every cold start. Every row below is a **citation, not a second decision** — where this register and an owner ever differ, the owner governs. The two owners are **[05](05-aspnet-core-migration-approach.md) §8.2 for the mechanism** — the store, the key, the lifetime, the invalidation contract it hands back to this register, the concurrency and the degraded behaviour — and **[05](05-aspnet-core-migration-approach.md)'s required-coverage table for the acceptance criteria**. Deliverable [06](06-azure-hosting-recommendations.md) §6.4 owns neither: its register row **S7 is the sizing consequence only**, and says so in terms — the two extra rows the chosen store adds to `dbo.SessionCache`, with the mechanism cited to 05 rather than restated.

| Element | Owner | Where it is stated in full |
| --- | --- | --- |
| **Key** | One **global versioned key** — the value at `genre-menu:{stamp}` with the stamp itself held at `genre-menu:stamp`, rotated after a committed write so that every replica's next read resolves to the new value. One key for every user, because the aggregate is identical for all of them — a per-user or per-request key would cache nothing while appearing to | [05](05-aspnet-core-migration-approach.md) §8.2, the caching mechanism |
| **Store and lifetime** | The **SQL-backed distributed cache** — the `dbo.SessionCache` table session already uses — with the **absolute 60-second** expiration of the paragraph above. Shared rather than per instance, which is what makes a stamp rotation reach every replica; [06](06-azure-hosting-recommendations.md) §6.4 owns the sizing consequence of the two extra keys | [05](05-aspnet-core-migration-approach.md) §8.2, the same mechanism |
| **Invalidation** | Any **committed** write to `Albums` or `OrderDetails` — all four paths of the table above, including the administrator `GenreId` edit. After the commit, never before it: a stamp bumped ahead of a transaction that then rolls back would publish an ordering the database never held | **This register**, §5.2.1 above; executed by the port |
| **Concurrency** | **Per-key single-flight**, so a cold instance under concurrent load issues **exactly one** database read rather than one per in-flight request. Without it, the nested aggregate is at its most expensive precisely when the instance is busiest | [05](05-aspnet-core-migration-approach.md) §8.2, the caching mechanism |
| **Failure** | Compute the aggregate directly and render normally when the cache is unreachable, logging once per occurrence-window rather than per request. The component renders from the shared layout on every page, so a failure in a navigation aggregate must **never** become a 500 | [05](05-aspnet-core-migration-approach.md) §8.2, its three-case degraded contract |
| **Acceptance** | Exactly **one** database read on a cold miss under N concurrent requests, **zero** on a warm hit, one after the expiration elapses, and one after each invalidation event — criteria a no-cache implementation **fails** | [05](05-aspnet-core-migration-approach.md), its required-coverage table for the behaviour suite |

| | |
| --- | --- |
| **Remediation** | **Cache the genre aggregate under the complete contract of §5.2.1 and §5.2.2** — this register's invalidation rule covering **all four write paths**, together with the versioned key, the shared distributed store, the absolute 60-second lifetime, the per-key single-flight and the never-a-500 degraded contract that deliverable [05](05-aspnet-core-migration-approach.md) §8.2's caching mechanism owns, and the cold-miss/warm-hit read counts that deliverable [05](05-aspnet-core-migration-approach.md)'s required-coverage table owns as acceptance. And scope the cart read so it is not recomputed for pages that do not display it — in MVC 5 and MVC 4 that also means eliminating the double enumeration and the per-row lazy load by projecting the titles in the query. Both remain behaviour-preserving. The mechanism by which the child actions become view components is deliverable 05's; this entry is about the query cost and the correctness of the invalidation rule, both of which survive the conversion unless they are addressed deliberately. |
| **Owner** | Performance |

### 5.3 F-08-04 — Unbounded result sets, and eight repeated tracked lookup reads (all three editions)

| | |
| --- | --- |
| **F-08-04** | **Two list actions materialize an entire table with no paging or projection, and the administration controller re-reads two whole lookup tables — tracked — on every one of its four form actions** |
| **Severity** | **Medium**, set by the two list actions: bounded today by the seeded catalog size, unbounded in principle. The eight lookup reads of §5.3.1 are **Low** in their own right — a per-request cost and a tracking defect rather than an unbounded one — and are recorded inside this entry rather than as a separate finding because they are the same shape of query at the same site. |
| **Editions** | All three; the code is byte-identical between MVC 4 and MVC 5 per section 3.1, and MVC 3 carries the same two sites and the same eight reads at its own line numbers |

`StoreController.Index` materializes every genre [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:18]. The administration list does the same for albums, with two eager `Include`s and an `OrderBy`, then `ToList()` [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:22-24] — 462 albums as seeded, each with its genre and artist loaded.

#### 5.3.1 The eight `SelectList` lookup reads, enumerated

An inventory that stops at the two list actions understates the administration controller's read volume by a factor of four, so the eight remaining reads are named here with their sites. Each of the four form actions builds two drop-down lists, and each list is constructed from a `DbSet` directly — `new SelectList(db.Genres, "GenreId", "Name")` and the `db.Artists` equivalent — so each is an unbounded, **change-tracked** read of a whole lookup table whose rows the action never modifies.

| Action | Reads | Site (MVC 5) | Same site in MVC 4 | Same site in MVC 3 |
| --- | --- | --- | --- | --- |
| `Create` (GET) | `Genres`, `Artists` | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:45-46] | [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:45-46] | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:40-41] |
| `Create` (POST, invalid model) | `Genres`, `Artists` | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:63-64] | [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:63-64] | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:58-59] |
| `Edit` (GET) | `Genres`, `Artists` | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:78-79] | [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:78-79] | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:69-70] |
| `Edit` (POST, invalid model) | `Genres`, `Artists` | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:95-96] | [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:95-96] | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:86-87] |

**Eight reads per edition, and they are repeated rather than merely numerous.** The two POST rows execute only on the invalid-model path, so a single valid create or edit performs two of the eight; an administrator correcting a validation error performs four. Two properties make them worth an inventory row rather than a footnote:

- **They are change-tracked for no reason.** A `SelectList` needs identifier and display values and never writes to the entities behind them, so every genre and artist entering the change tracker is overhead the request cannot use — and it sits in the same context instance that the action then saves, per F-08-09.
- **They are unbounded like the two list actions.** MVC 5 seeds 15 genres and 303 artists and MVC 3 seeds 10 and 149, both counted in section 3.4 — small either way; the artist table is the one that grows with real use, and nothing in the query bounds it.

**The mechanics of the replacement read are this register's own, and are stated here rather than
routed.** No other deliverable specifies them — the port design owns the injection and lifetime change
that these reads sit inside (F-08-09), and it owns paging and projection for the *two list pages*, but
nothing anywhere carries a lookup-read or tracking-behaviour contract for the four administration form
actions. So this entry carries it, in four parts:

- **Projected, not entity.** Each read returns only the two values the drop-down renders — the
  identifier and the display name — rather than a `Genre` or an `Artist` instance. That is what the
  `SelectList` consumes today [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:45-46] and
  it is all it consumes.
- **Untracked.** The read must not place rows in the change tracker. This is the part with a
  correctness dimension rather than only a cost one: the same context instance is saved on these paths,
  so tracked lookup entities are attached state that nothing on the path intends to write.
- **Read at most once per lookup table per request, and reused.** **This does not reduce today's
  per-request read count**, and the entry does not claim it does: each of the four actions builds one
  list per table, so it performs one read per table already. The rule is stated as a **bound** so that
  a later refactor which renders these lists from more than one place — a shared editor partial, a
  re-render after a failed save — cannot silently multiply them.
- **Deterministically ordered.** A materialized list reused within a request must render the same
  option order every time it is rendered, so the read carries an explicit order — display name, then
  identifier as the tiebreaker. The source declares **no** order at all at these four sites, so this
  makes an unspecified order specified; it changes nothing the source defines, and the selected value
  in each drop-down is unaffected.

**Acceptance**, stated so a no-op implementation fails it: for each of the four form actions, **at most
one read per lookup table per request**, **no lookup entity in the change tracker** at the point the
action saves, and the rendered options and their selected values **unchanged** from the baseline. No
latency or throughput figure is asserted anywhere in this entry — this register has measured none, and
[07](07-effort-risks-sequencing.md) is where cost is expressed.

| | |
| --- | --- |
| **Remediation** | Paging or projection at the two list sites — the administration list is the one that grows with real use. For the eight lookup reads of §5.3.1: **projected, untracked, deterministically ordered lookups, bounded at one read per lookup table per request**, in the four parts §5.3.1 states and against the acceptance §5.3.1 states with them. Those mechanics are **this register's own remediation** — no other deliverable carries a lookup-read or tracking contract for these four actions — and they are behaviour-preserving: the rendered options and their selected values are unchanged. |
| **Owner** | The port, for both parts, executing §5.3.1's contract; **this register** owns that contract and its acceptance; performance consulted on the lookup reads |

### 5.4 F-08-05 — Unchecked query results, unevenly distributed (all three editions)

| | |
| --- | --- |
| **F-08-05** | **Four `Single` calls on unvalidated input, three unguarded `Find` results, and one guard that cannot be reached** |
| **Severity** | **Medium** — the failure mode is an unhandled exception on a crafted or stale request, and section 7.1 establishes that nothing records it |
| **Editions** | All three — **but not identically**: the guarded/unguarded distribution differs by edition, per the table below, and the "three of four guarded" pattern holds in two editions of the three |

The distribution matters, because most of this code is careful and an entry that flattened it would be unfair to the codebase. It is set out twice below, because two different flattenings are possible: **by edition**, where the guarded proportion differs, and **by site**, where the earlier count of this entry was short. Neither table repeats the other.

**The guarded/unguarded distribution, per edition.** This is the correction that matters most in this entry, because "three of four guarded" is a true statement about two editions and a false one about the third:

| Edition | Administration `Albums.Find` calls | Guarded | Locators |
| --- | --- | --- | --- |
| **MVC 5** | 4 | **3** — `Details`, `Edit`, `Delete` return `HttpNotFound()`; `DeleteConfirmed` does not check | `Find` at [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:32], [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:73], [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:105], [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:119]; the guards' results at [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:35], [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:76], [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:108] |
| **MVC 4** | 4 | **3** — the same three actions, at the same line numbers | `Find` at [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:32], [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:73], [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:105], [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:119]; the guards' results at [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:35], [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:76], [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:108] |
| **MVC 3** | 4 | **0** — no call is checked, and the edition contains **no `HttpNotFound()` call anywhere** | `Find` at [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:31] (`Details`), [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:68] (`Edit`), [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:96] (`Delete`), [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:106] (`DeleteConfirmed`) |

`git grep -c 'HttpNotFound()' -- 'src/MVC5/*.cs' 'src/MVC4/*.cs'` returns `3` for each; the same search over `src/MVC3/*.cs` returns nothing at all. MVC 3's two worst cases are worth naming rather than leaving inside a count: `Edit` dereferences the possibly-null result on the next line, `album.GenreId` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:69], and `DeleteConfirmed` passes it to `Albums.Remove` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:107]. Deliverable 12 records the same distribution from the blocker side, in [F-12-08](12-migration-blockers.md#f-12-08--three-httpnotfound-calls), where the absence is what makes MVC 3 have no not-found path to port.

**The full census in MVC 5, the migration source.** The earlier version of this entry counted three `Single` sites and one unchecked `Find`. The complete count is **four** input-reachable `Single` sites and **six** `Albums.Find` sites, three of the six unguarded — and two of the unguarded ones are outside the administration controller, which is where a reader of the previous count would not have looked:

| # | Site | Reached from | Guarded? | What happens with no row |
| --- | --- | --- | --- | --- |
| 1 | `Genres.Include("Albums").Single(g => g.Name == genre)` [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:30-31] | A query-string value, unvalidated | No | Throws — the only site where **more than one** row is also possible (see below) |
| 2 | `Albums.Single(album => album.AlbumId == id)` [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:37-38] | A route value, in `AddToCart` | No | Throws |
| 3 | `Carts.Single(item => item.RecordId == id).Album.Title` [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:61-62] | The AJAX request body, in `RemoveFromCart` | No | Throws; the read is also unscoped to the caller's cart, which deliverable 09 owns as a security finding rather than as debt |
| 4 | `Carts.Single(cart => cart.CartId == ShoppingCartId && cart.RecordId == id)` [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:64-66] | The same AJAX id, one layer down | No — see the unreachable guard below | Throws |
| 5 | `Albums.Find(id)` [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:32] | Administration `Details` | **Yes** [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:35] | `HttpNotFound()` |
| 6 | `Albums.Find(id)` [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:73] | Administration `Edit` | **Yes** [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:76] | `HttpNotFound()` |
| 7 | `Albums.Find(id)` [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:105] | Administration `Delete` | **Yes** [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:108] | `HttpNotFound()` |
| 8 | `Albums.Find(id)` [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:119] | Administration `DeleteConfirmed` | No | `null` passed to `Albums.Remove` [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:119-120] |
| 9 | `Albums.Find(id)` [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:38] | **The public album-detail page**, a route value | No | `null` model passed to the view [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:40], which then dereferences it while rendering |
| 10 | `Albums.Find(item.AlbumId)` [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:134] | **Order creation**, per cart line | No | `null` dereferenced at `album.Price` [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:140], inside the checkout write |

**One guard reads as protection and provides none.** `RemoveFromCart` tests `if (cartItem != null)` [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:70] on the result of the `Single` at [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:64]. `Single` never returns `null` — it throws when the row is absent — so the guarded block is entered on every path that reaches it and the `else` case it implies is **unreachable**. This is worth recording separately from the missing guards, because it is the more dangerous shape: a reviewer scanning for null checks finds one here and moves on.

**Remediation — and the two failure modes must not be collapsed, because one operator change fixes only the first.**

| Failure mode | Where it applies | What actually fixes it |
| --- | --- | --- |
| **Zero rows** | All four `Single` sites (1-4), and the three unguarded `Find` sites (8, 9, 10) | `SingleOrDefault` at sites 1-4 — `Find` already returns `null` — followed by an **explicit** not-found result at sites 1, 2, 3, 4, 8 and 9, and by a decision at site 10, where a missing album during order creation is a checkout failure rather than a page-level 404. Sites 5, 6 and 7 need **no** remediation: they already return a not-found result, and their only change is the result type, which deliverable 12's [F-12-08](12-migration-blockers.md#f-12-08--three-httpnotfound-calls) owns. The `if (cartItem != null)` guard at [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:70] becomes meaningful once [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:64] stops throwing |
| **More than one row** | **Site 1 only.** `Genre.Name` is a plain `string` property with no key and no unique constraint declared on the model [src/MVC5/MvcMusicStore/Models/Genre.cs:8], so two genres may share a name. The other three `Single` predicates all include a primary key — `Album.AlbumId`, and `Cart.RecordId`, which carries `[Key]` [src/MVC5/MvcMusicStore/Models/Cart.cs:8-9] — so at most one row can match | **Not `SingleOrDefault`.** That operator throws on a duplicate exactly as `Single` does; changing it hides the empty case and leaves this one. A duplicate is a **data-integrity condition**, not a request error: it must be detected, logged with the offending value, and surfaced as a server error rather than as a not-found, and the durable fix is a uniqueness constraint on the column. Substituting the operator and declaring the site handled is the specific mistake this row exists to prevent |

**Why this stays Medium.** The consequence at every site is an unhandled exception — a 500, or a rendering failure — with no data corruption and no authorization bypass, on input that is crafted, stale, or a deleted-album race. Two aggravating interactions are real but are owned elsewhere at their own severities: site 10 fails inside the order write, where the bare `catch` of F-08-08 (High) discards the exception and redisplays the form, and F-08-13 (Critical) means nothing anywhere records that it happened. Medium is the consequence of *this* entry; the invisibility of the failure is charged to those two.

#### 5.4.1 `SingleOrDefault` closes three of the four `Single` sites and only half of the fourth

**What the operator does, stated plainly, because the difference is the whole of this sub-section.**
`SingleOrDefault` converts the **zero-row** case into a `null` the caller can turn into a not-found
result. It does **not** convert the multiple-row case: with more than one matching row it **still
throws**, for the same reason `Single` does. So the substitution is a **complete** fix wherever the
predicate can match at most one row, and a **half** fix wherever it can match more — and a remediation
that recorded it as a fix for "an unknown or duplicated genre" would be claiming something the operator
does not do.

**Where it is complete.** **Three of the four** `Single` sites of the census above — sites 2, 3 and 4 —
filter on a key: site 2 on `Album.AlbumId` [src/MVC5/MvcMusicStore/Models/Album.cs:10], and sites 3 and 4 on the
`[Key]`-annotated `Cart.RecordId` [src/MVC5/MvcMusicStore/Models/Cart.cs:8-9] — so the multiple-row case
cannot arise at any of them and an explicit not-found result answers each one entirely. The same is true
of every `Albums.Find` site, sites 5 to 10, because `Find` takes a key by definition: what the three
unguarded ones need is a null check rather than an operator change, `Find` already returning `null`
where `Single` throws.

**Where it is not.** `Browse` filters on `g.Name == genre`
[src/MVC5/MvcMusicStore/Controllers/StoreController.cs:30-31] against a property carrying **no key, no
unique constraint and no annotation of any kind** [src/MVC5/MvcMusicStore/Models/Genre.cs:8]; the
entity's key is `GenreId` [src/MVC5/MvcMusicStore/Models/Genre.cs:7]. Two facts make duplicate names a state the target may legally hold
rather than a hypothetical:

- **The source model establishes no uniqueness on the name**, and this assessment does not treat the
  source *schema* as established either — it is evidence rather than proof until the extraction runs
  [12 §5](12-migration-blockers.md), and F-08-12 records that the migration source ships no schema
  script to check against.
- **The migration deliberately preserves such duplicates.** Deliverable
  [05](05-aspnet-core-migration-approach.md)'s duplicate-detection rule names `Genre.Name` explicitly
  among the candidate natural keys the load **does not reject on** — duplicates there are counted,
  enumerated by key in a controlled artifact and carried into the target as they stand, because
  rejecting on an undeclared natural key would reject legitimate historical rows. So the browse path has
  to be correct **in the presence of** duplicates, not merely correct once someone assumes there are
  none.

**The resolution contract belongs to deliverable 05, and this register does not author a second one.**
What the browse path does in the presence of duplicate genre names is a migration-and-interface
decision rather than a debt measurement, so under the one-fact-one-owner rule of §1.2 it is stated in
full in [05](05-aspnet-core-migration-approach.md) — the **duplicate-`Genre` browse contract** — and
cited here. What this register keeps is what only it carries: the measured source behaviour above, its
throw-on-duplicate consequence, the severity, the direction of remediation and the owner.

**Three properties of that contract are named here, because the debt is not provably closed without
them.** They are named and cited, not restated: [05](05-aspnet-core-migration-approach.md) holds the
branch mechanics, the migration stage the schema change belongs to, the routing form and the test cases.

- **It is non-lossy unconditionally.** The migration **merges no `Genre` row, deletes no `Genre` row and
  renames no `Genre` value.** Merging, deleting, renaming, and breaking a tie by assigning a distinct
  name, are therefore **not** dispositions this register offers, and not one of the four is non-lossy:
  merging or deleting removes a catalog row the source holds; renaming rewrites a value visitors read,
  and one that both internal links are built from [src/MVC5/MvcMusicStore/Views/Store/Index.cshtml:13],
  [src/MVC5/MvcMusicStore/Views/Store/GenreMenu.cshtml:9-11]; and asserting uniqueness on the name
  before the extraction has established the target column's collation can fail, or fold two rows
  together, under equality semantics that have not yet been configured.
- **It is gated on the schema extraction, and both of its branches are specified there.** Whether a
  duplicate `Genre.Name` exists under the target's configured collation is a *result* the extraction
  produces, never a premise either document may assume: F-08-12 records that the migration source ships
  no schema script to check against, and [12 §5](12-migration-blockers.md) records the extraction as the
  gate. One branch applies where the extraction finds no duplicate name and one where it finds
  duplicates; this register selects neither and pre-empts neither.
- **The duplicates branch is a conditional approved delta, and it is governed as one.** Its approval
  owners are **Product and Data**, and it carries its own coverage. Both records live in
  [05](05-aspnet-core-migration-approach.md) — the approved-delta table and the required-coverage table
  it owns — and **this register adds no row to either.** That is exactly why it must not propose a
  data-changing or URL-changing disposition of its own: a disposition invented here would reach
  implementation with no approval owner, no delta row and no test, which is the failure mode the
  one-fact-one-owner rule exists to prevent.

**First-match-wins remains explicitly rejected**, and that rejection is this register's to keep, because
it is the reason the naive fix is wrong rather than merely incomplete. Silently taking one of two
matching genres makes the other permanently unreachable, and it does so with no exception, nothing in
the source to indicate it happened, and — per section 7.1 — nothing anywhere that would record it.

**Acceptance for this finding, which is two cases and not one.** The browse path must be exercised with
**zero** matching genres and with **more than one**, and neither may produce an unhandled exception. The
zero-row case is closed at this site by `SingleOrDefault` and an explicit not-found result — the
remediation direction this entry states for all **four** `Single` sites. The multiple-row case is answered
by whichever branch [05](05-aspnet-core-migration-approach.md)'s contract selects, so **its coverage row
is 05's to place**, in the required-coverage table 05 owns: this register does not place it, does not add
a coverage row of its own, and does not assume one is already there. What this entry fixes is that the
multiple-match case exists as a case at all and is not answered by an operator substitution that
still throws. The **three** key-predicate `Single` sites and the **three** unguarded `Find` sites need
their not-found paths exercised for the same reason at a lower severity, site 10 among them, where the
missing row is met inside the checkout write rather than on a page.

| | |
| --- | --- |
| **Remediation** | **The ten-site census above is the authority for this remediation, and every site it lists is named here.** Seven sites need a change, and they need three different ones. **(a) `SingleOrDefault` plus an explicit not-found result at all four `Single` sites** — sites 1 to 4, at [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:31], [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:38], [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:62] and [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:64] — the not-found result being the point, since a bare `null` merely moves the throw. **(b) A null check at the three unguarded `Find` sites**, which need no operator change because `Find` already returns `null`: `DeleteConfirmed` [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:119], whose unchecked result reaches `Albums.Remove` [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:120]; the public album-detail page [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:38], whose null model reaches the view [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:40]; and order creation [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:134], whose null is dereferenced at `album.Price` [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:140] inside the checkout write, where a missing album is a checkout failure and not a page-level 404. **(c) Remove the unreachable guard's false assurance** at [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:70], which reads as a null check over a `Single` that throws instead of returning null and becomes meaningful only once [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:64] stops throwing. **Sites 5, 6 and 7 need nothing here** — the three guarded administration `Find` calls at [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:32], [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:73] and [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:105], already checked at [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:35], [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:76] and [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:108]; their only change is the result type, which deliverable 12's [F-12-08](12-migration-blockers.md#f-12-08--three-httpnotfound-calls) owns. **Site 1 needs more than the operator substitution**, and §5.4.1 states what: it is the only site where more than one row is possible, so `SingleOrDefault` closes its zero-row case and leaves a duplicated genre name throwing, and that residue is a **data-integrity condition** — detected, logged with the offending value and surfaced as a server error — answered by the **duplicate-`Genre` browse contract** that deliverable [05](05-aspnet-core-migration-approach.md) owns and §5.4.1 cites: non-lossy unconditionally, gated on the schema extraction, its duplicates branch governed as a conditional approved delta in 05's own delta and coverage tables. This register proposes no merge, deletion, rename or uniqueness disposition of its own, and first-match-wins stays rejected. **The target result contracts** — which result type each site returns, what the AJAX endpoint returns, and what the checkout does when an album vanishes mid-order — are [05](05-aspnet-core-migration-approach.md)'s; what this register owns is the census, the two failure modes and the direction. |
| **Owner** | The port, for the changes at all seven sites that need one — the four `SingleOrDefault` substitutions with their not-found results, the three `Find` null checks, and the unreachable guard's removal — including the browse site's zero-row not-found result; **05** for the target result contracts at every site, and for the duplicate-`Genre` browse contract that answers site 1's multiple-row case, including the approval owners and coverage its conditional branch carries; **12** ([F-12-08](12-migration-blockers.md#f-12-08--three-httpnotfound-calls)) for the result-type change at the three already-guarded sites |

### 5.5 F-08-06 — Anti-forgery validation covers one controller of the four that need it

| | |
| --- | --- |
| **F-08-06** | **Five state-changing POST actions accept requests with no anti-forgery validation, and one state-changing action is a `GET`** |
| **Severity** | **High** — cross-site request forgery against album administration, cart removal and order placement |
| **Editions** | MVC 5 and MVC 4 both leave 5 of their state-changing POSTs unvalidated; **MVC 3 validates nothing anywhere** |

**MVC 5, measured.** Thirteen POST actions exist across the six controllers. All eight in `AccountController` carry `[ValidateAntiForgeryToken]` [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:55], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:88], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:113], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:147], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:199], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:236], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:264], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:301]. None of the other five does:

| Unvalidated state-changing POST | Locator | Effect |
| --- | --- | --- |
| `StoreManagerController.Create` | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:53] | Inserts an album |
| `StoreManagerController.Edit` | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:86] | Updates an album |
| `StoreManagerController.DeleteConfirmed` | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:116] | Deletes an album |
| `ShoppingCartController.RemoveFromCart` | [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:54] | Removes a cart line and commits |
| `CheckoutController.AddressAndPayment` | [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:25] | Writes an order and empties the cart |

**One state-changing action is a `GET` and no anti-forgery policy can cover it.** `AddToCart` declares no verb attribute, loads the album [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:37-38], mutates the cart [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:43] and commits [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:45] before redirecting [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:48] — the whole action spanning [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:33-49]. Token emission is not the gap: ten MVC 5 views emit `@Html.AntiForgeryToken()` (`git ls-files 'src/MVC5/*.cshtml' | xargs grep -l AntiForgeryToken | wc -l` → `10`). **Emission is not validation.**

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

`StoreManagerController` has **three** POST actions [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:53], [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:86], [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:116]. A register reporting two would understate the exposed surface by one delete action.

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
- **The store the credential provisions into is itself committed — in the two editions that provision.** F-08-11 records fourteen tracked database binaries, and its table locates the two that matter here: MVC 5's ASP.NET Identity 1.0 pair and MVC 4's SimpleMembership pair, both under the applications' own `App_Data`. In those two editions the provisioned account can therefore exist in tracked data as well as in tracked configuration. **The scope stops there.** MVC 3 has no provisioning path, so no provisioned account is attributable to it, and the credential-shaped database committed under its *tutorial assets* is neither its configured store nor evidence of a provisioned account — F-08-11 records what that file is and is not.

| | |
| --- | --- |
| **Remediation** | Remove the credential from source entirely and provision the administrator through an operator-invoked command that takes the secret from a non-persistent channel. The mechanism, the audit record and the idempotence requirements are specified by the migration approach; the security analysis is deliverable 09's. **This register does not repair it** — see section 1.3. |
| **Owner** | Security |

### 5.7 F-08-08 — Swallowed checkout errors (all three editions)

| | |
| --- | --- |
| **F-08-08** | **The order-writing transaction is wrapped in a bare `catch` that discards the exception** |
| **Severity** | **High** — combined with section 7.1, a failed order leaves no trace anywhere |
| **Editions** | **All three.** MVC 5 and MVC 4 are the byte-identical file (section 3.1); MVC 3's `CheckoutController` differs by 12 diff lines but carries the same bare `catch` around the same write |

The entire order write — add order, create order details from the cart, commit [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:44-51] — is enclosed in `catch` with **no exception variable** [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:58], which redisplays the view [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:61] and records nothing. The customer sees the form again; the operator sees nothing at all.

**MVC 3 has the same defect and is counted here rather than noted beside it**, because a bare `catch` at [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/CheckoutController.cs:56] discards the exception identically. What differs in MVC 3 is only the *transaction shape* it discards the exception from: its cart owns its own context and commits internally at five points (section 3.4), so a failure mid-sequence can leave a partially-committed order where MVC 5's single `SaveChanges` cannot. That makes MVC 3's instance worse rather than absent, and the remediation below covers all three:

```bash
git grep -n 'catch' -- 'src/*/Controllers/CheckoutController.cs' 'src/*/*/*/Controllers/CheckoutController.cs'
# -> MVC3 CheckoutController.cs:56, MVC4 :58, MVC5 :58 — all bare
```

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

`StoreManagerController` disposes the context it constructed [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:125-128], and MVC 5's `AccountController` disposes the `UserManager` built by its own chained constructor [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:324-331]. The construction sites they pair with are field initializers and an ad hoc instance inside a method — [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:15], [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:11], [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:12], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:19] and [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:32].

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

The seed being rebuilt is **826 LF — 827 content lines** — of hardcoded C# [src/MVC5/MvcMusicStore/Models/SampleData.cs:1-827]. That is the physical-line metric; **820** is the same file by the non-blank sizing metric, and the two are named separately here rather than in one figure, per deliverable 01 §2.4. It is F-08-01's largest single file and 39 percent of the migration source by section 4.2.

| | |
| --- | --- |
| **Remediation** | Replace automatic destructive initialization with versioned schema change applied at deployment time, and make any seeding an explicit, guarded, non-production action. The mechanism, the guard design and the deployment ordering are owned by deliverables 05 and 06; the migration of the existing data is a workstream deliverable 03 sequences. |
| **Owner** | The data workstream |

**This entry is the debt closure for deliverable 09's finding F-09-31**, and the identifier is printed here because 09 §8.3 requires that register's Consumers column to be checkable from this end as well as from its own: a reader arriving from the row that names this deliverable has to land on a named entry, not on a topic. F-09-31 is the same construct at the same three locations, carried there as **Critical** on the security framing — destruction of customer data triggered by an ordinary deployment, needing no attacker (09 §6.7) — and its Consumers cell reads `05, 06, 08`. What this register adds is the debt view and nothing else: the severity, the remediation and the owner stated above, with the replacement mechanism left to deliverables 05 and 06 and the migration of the existing data to deliverable 03. The register's only other row directed here, **F-09-34**, closes at F-08-11 in §6.2, and those two are the whole of what 09 routes to this deliverable.

### 6.2 F-08-11 — Fourteen database binaries committed, 43,376,640 bytes, including two credential stores and one credential-shaped tutorial artifact

| | |
| --- | --- |
| **F-08-11** | **Live database files — catalogs, two credential stores and one unreferenced credential-shaped database — are tracked in version control** |
| **Severity** | **High** — tracked credential material and customer data, permanent repository weight, and a merge hazard on any binary that is ever opened |
| **Editions** | All three. **Two of the three credential-shaped databases are their edition's own declared store; MVC 3's is an unreferenced tutorial artifact that no tracked configuration names** — the qualification below states what that does and does not establish, and the severity is unaffected by it |
| **Remediation** | Untracking these files does not remove them from history; only a history rewrite does, and that is a decision with consequences for every fork and clone. The choices are (a) accept the history and untrack going forward, or (b) rewrite. Either way, no environment should ever attach a tracked file — the setup guidance for this repository already requires serving a copy outside the checkout so SQL Server cannot write to tracked binaries, which is a mitigation, not a fix. Beyond that, the two credential-bearing application stores in history — MVC 4's and MVC 5's — should be treated as compromised, because F-08-07's committed credential provisions into them and the material is therefore known to be there. The tutorial `ASPNETDB.MDF` should be handled the same way, but as a **conservative precaution, pending content verification**: what is established is that a credential-shaped, unreferenced database is tracked, not what accounts it holds, so precautionary handling is the correct response while an assertion that its credentials are exposed is not one this register can make. Deliverable [09 §6.9](09-security-assessment.md) carries the same qualification and owns the exposure analysis. It is also the one member of the set whose removal has **no runtime consequence to weigh**, because no configuration in the repository opens it (§6.2.1) — the tracked-file exposure follows from the file being in history, not from which application reads it, so the unresolved question of MVC 3's runtime store does not gate any of this. |
| **Owner** | The repository owner (history decision); security (credential-store exposure) |

```bash
git ls-files | grep -icE '\.(mdf|ldf)$'                                    # -> 14
git ls-files | grep -iE  '\.(mdf|ldf)$' | xargs -d '\n' stat -c '%s' \
  | awk '{s+=$1} END {print s}'                                           # -> 43376640
```

`awk` performs the sum because `bc` is not installed on this host.

| Location | Files | What they are |
| --- | --- | --- |
| `src/MVC3/MvcMusicStore-Assets/Data/` | 4 | The tutorial catalog pair plus `ASPNETDB.MDF` and its log — an **unreferenced, credential-shaped tutorial artifact** carrying a classic ASP.NET Membership schema, and at 10,485,760 bytes the largest single file in the set. **No tracked configuration names it**, and this register infers no accounts, hashes or provider selection from it; see below |
| `src/MVC4/MvcMusicStore/App_Data/` | 6 | The catalog pair, the SimpleMembership credential pair **declared by the edition's own `DefaultConnection`** [src/MVC4/MvcMusicStore/Web.config:12-17], **and an unreferenced scratch pair** |
| `src/MVC5/MvcMusicStore/App_Data/` | 4 | The catalog pair and the ASP.NET Identity 1.0 credential pair, both **declared by the edition's own connection strings** [src/MVC5/MvcMusicStore/Web.config:12-13] |

**MVC 3's credential-shaped file is an unreferenced tutorial artifact, and this register assigns it to no application.** It sits under `src/MVC3/MvcMusicStore-Assets/Data/`, the tutorial payload, not under the completed application — which has **no `App_Data` directory at all**:

```bash
git ls-files 'src/MVC3/MvcMusicStore-Completed/*' | grep App_Data | wc -l   # -> 0
git ls-files 'src/MVC3/MvcMusicStore-Assets/Data/*'
# -> ASPNETDB.MDF, MvcMusicStore-Create.sql, MvcMusicStore.mdf,
#    MvcMusicStore_log.ldf, aspnetdb_log.ldf
```

The completed application's `web.config` enables `<roleManager enabled="true" />` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:15] and Forms authentication [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:26], but declares **no membership provider, no role provider and no `LocalSqlServer` connection string** — its only connection string is the SQL Server Compact catalogue entry [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:55-59]. Classic `Membership` and `Roles` therefore resolve through whatever the host's **machine-level** ASP.NET configuration supplies, against the host's own connection-string setting — a property of the host rather than of this repository. **Which provider that is, and which database it resolves to, is not determinable here**, so this register names neither. Deliverable 01 §8.3 owns that architectural fact and deliverable 10 §10.2 owns the database consequence, **retaining the store's identity as a question to be settled by verifying the machine-level provider and connection string on the supported Windows runtime before the requirement is stated as final.** This register therefore records what is certain — a tracked binary carrying a credential schema — and does **not** assert that the completed application reads it.

**The severity does not depend on that question.** A credential-shaped database file in version control is exposure whichever application, if any, ever opened it, and none of the four MVC 3 files is covered by an ignore rule at all (section 10.7). What is undetermined is *which store the edition resolves to*, not *what schema this file carries*.

**The scratch pair is debt in its own right.** `src/MVC4/MvcMusicStore/App_Data/MvcMusicStore-work.mdf` and `src/MVC4/MvcMusicStore/App_Data/MvcMusicStore_log-work.ldf` are referenced by no tracked source, configuration, project or solution file — 4,259,840 bytes of committed working copy:

```bash
git ls-files '*.cs' '*.config' '*.cshtml' '*.csproj' '*.sln' | grep -v /packages/ \
  | xargs grep -il 'MvcMusicStore-work' | wc -l                           # -> 0
```

**Ten of the fourteen are ignored by the repository's own rules and tracked anyway — and the other four are not ignored at all.** This distinction matters, and collapsing it would misstate the finding. `.gitignore:32` declares `App_Data/`, which covers the MVC 4 and MVC 5 files; the four MVC 3 files sit under `Data/`, which no rule matches. So ten are gitignored-yet-tracked — a rule added after the files were already in the index, which cannot untrack them — and four are simply tracked, with no rule ever expressing an intent to exclude them. Section 10.7 records the correct probe and the exact outputs.

#### 6.2.1 `ASPNETDB.MDF` is an unreferenced, credential-shaped tutorial and schema-evidence asset that this repository assigns to no application

The classification matters more than it looks, and it cuts both ways. An inventory row that calls a committed binary an edition's runtime credential store turns repository weight into a stated runtime dependency, and a migration reading that row would plan to move a database no application is configured to open. A row that declares it *categorically not* the runtime store overreaches in the other direction, because MVC 3's effective store is the host's and nobody has read the host. The correction is therefore narrow and evidenced: what the repository settles is that **nothing here names this file**, and it settles nothing about what the running edition resolves to.

**Nothing in the repository references it.** The probe is repository-wide and its result is zero:

```bash
git grep -il 'ASPNETDB' -- 'src/'   # -> exit 1, no matching file
```

**And MVC 3 declares no store for it to be.** `<roleManager enabled="true" />` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:15] and Forms authentication [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:26-28] are both on, but the file declares **no** `<membership>` element, **no** `<providers>` collection and **no** `LocalSqlServer` connection string — its only connection string is the SQL Server Compact catalog one [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:55-59]. So the credential store MVC 3 resolves to at runtime is **inherited from the host's machine-level configuration**, and its identity — the provider actually selected there, and the connection string it resolves — **is unknown from this repository and remains unverified.** Deliverable 10 §10.1 and §10.2 own the per-edition topology and state it correctly; deliverable 10 §13.2 keeps that verification open as item 2, to be answered on a supported Windows runtime rather than guessed at here. **This register does not guess in either direction: it names no runtime store for MVC 3, and it infers no accounts, no password hashes and no provider selection from any committed file.**

**What the file is, positively, in three parts:**

- **Tutorial payload.** It sits in the tutorial asset tree, and the tree's own note describes the directory as carrying "a database (only used if you won't be using SQL Server CE)" [src/MVC3/MvcMusicStore-Assets/readme.txt:6] — a supplied alternative for a reader working through the tutorial, not a component of the completed application, which ships no `App_Data` directory of its own.
- **Schema evidence.** It is the repository's only artifact carrying the classic ASP.NET Membership schema, and deliverable 09 §6.9 uses a printable-string probe of it as such. That is evidence of a schema, not proof of a deployment, and it is the only role in which a downstream deliverable may cite it.
- **A committed, credential-shaped database all the same.** Whatever it was supplied *for*, it is a Membership-schema database in version control, 10,485,760 bytes with a 516,096-byte log alongside it. Its exposure is real and independent of which application ever opened it — and it is an exposure stated on the file's schema, not on any account this register claims to have read out of it.

**Both debt categories therefore apply, and neither is a substitute for the other:**

| Category | The debt | Where it is owned |
| --- | --- | --- |
| **Repository debt** | 11,001,856 bytes of committed binary weight for an artifact no configuration references, and — per §10.7 — one of the four database binaries **no ignore rule matches at all**, so nothing in the repository ever expressed an intent to exclude it | The repository owner, as part of the history decision in this entry's remediation |
| **Security debt** | A credential-shaped database whose Membership schema carries password and salt columns, tracked in history on the same footing as the other two even though it is unreferenced. **What accounts it holds is not established**, and no entry here asserts any | Security; deliverable 09 F-09-34 owns the exposure analysis |

The reclassification changes no quantity in this entry: the set is still **14 files** and **43,376,640 bytes**, and the location table above still counts four files under `src/MVC3/MvcMusicStore-Assets/Data/`. What changes is what the row asserts about MVC 3's runtime.

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

There is no logging abstraction, no logging framework, no `TraceSource`, no ASP.NET health-monitoring configuration, no health endpoint and no metric of any kind. Error display is not observability either: the `customErrors` **name token** — the unit deliverable [09](09-security-assessment.md) §6.10 owns, which is not the same as an element — appears 24 times across the six XDT transform files, and **every occurrence is inside a comment block**, so no edition configures it live. 09 carries the breakdown: of those 24 tokens only 12 carry a leading angle bracket and only **6** are an element's opening tag, one commented example per file; the rest are prose references. All three units are zero live.

```bash
git ls-files '*.config' | grep -v /packages/ | xargs grep -h 'customErrors' | wc -l    # -> 24
# each of the six Web.Debug.config / Web.Release.config files: 4 name tokens on 4
# distinct lines, all inside <!-- ... --> template blocks (verified by an awk
# comment-block pass, section 14). Per file those four are: a prose <customErrors>
# reference, a prose "customErrors section" mention, the commented example element's
# opening tag, and that element's closing tag.
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

`.gitignore` anticipates publish profiles at `PublishProfiles/` [.gitignore:18] and build output at `build/` [.gitignore:29], and neither directory exists in the current checkout. The tracked history says the same thing, and because that is a claim about history it carries a history command: no publish-profile path has been added on any ref in this clone.

```bash
git log --all --diff-filter=A --name-only --pretty=format: -- '*.pubxml' '*.pubxml.user' | sort -u
git log --all --diff-filter=A --name-only --pretty=format: -- '*PublishProfiles/*' 'build/*' | sort -u
# -> both produce no output: no such path was ever added on any ref in this clone
```

What build logic the repository does have lives inside the MSBuild project files and MVC 4's `.nuget/NuGet.targets`, which deliverable 02 §5 inventories and deliverable 10 owns as build requirements.

This entry compounds F-08-16 and F-08-17: warnings are not errors, no analyzer runs, and there is no pipeline that would fail if either changed. Deliverable 02 §8.2 records the same shape on the dependency side, and it is the same absence rather than a comparable one: NuGet's restore-time audit reports against the resolved pins, but at the client's default audit level its warnings are advisory rather than fatal — so a restore that raises them still exits `0` — and no build, tooling or CI artifact in this repository records that output, retains it or gates on it. Deliverable 02 asserts no advisory identifier, severity or count anywhere, and keeps no dated copy of the output either (02 §8.1 carries the aged-pin class with the exact pins, 02 §8.2 the reasoning and the route a reader runs to obtain today's set, 02 §8.3 the scan the implementation phase owes). The consequence for this register is precise: the dependency exposure is reviewable only by someone who chooses to run that route, which is exactly the property this entry records about compiler warnings — observable output that nothing keeps and nothing fails on.

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

The migration consequence is concrete, and it is stated against **two different named units**, because collapsing them is how a view inventory goes wrong. Deliverable 01 §2.5 counts the **29** Razor files in MVC 5. Deliverable 05 owns how those 29 divide, and it divides them two ways that must not be substituted for each other:

- **Six** of the 29 are **source views naming a removed API or type** — the broad unit. Five are found by a legacy *namespace or type* search; the sixth, `Account/_ExternalLoginsListPartial.cshtml`, is missed by that search because its removed members are OWIN rather than Identity, and deliverable 05 §8.4 owns both the correction and the command that finds all six.
- **3 + 5 + 21 = 29** is the **disposition partition** — three views become view components' `Default.cshtml`, five need per-line work, and twenty-one port mechanically. Deliverable 05 §8.3 and §8.4 own it. The **five** here is a *per-line work* count, not the removed-API count, and the two differ by exactly the one view that is counted as a component rather than a per-line rewrite.

A build today reports nothing about any of the 29, so the port cannot lean on the compiler to enumerate what breaks — it has to be read, against whichever of the two units the reader is actually using.

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

So warnings are produced and then ignored: no `TreatWarningsAsErrors` in any configuration of any project, no `.ruleset`, no `.editorconfig`, no `Directory.Build.props` to apply a policy centrally, no analyzer package in any manifest — deliverable 02 §8.2 verifies that last point across all three `packages.config` files — and, per F-08-14, no pipeline that could fail on a warning even if a policy existed. **The evidence stops at the checkout, and the claim stops with it:** the current checkout tracks no build log or warning record of any kind (`git ls-files | grep -ciE '\.(log|binlog)$'` → `0`), so this register states no warning count for any edition. Nor could it: MVC 5's build status is **blocked pending a Windows verification run** ([10](10-build-and-deployment-requirements.md)), which owns every per-edition build outcome.

| | |
| --- | --- |
| **Remediation** | Establish the policy on the target project: analyzers enabled, a shared style configuration, and warnings escalated in the pipeline. Escalating warnings on the legacy projects is out of scope and would be a code change. |
| **Owner** | The port, with operations and platform for enforcement |

### 8.3 F-08-18 — A committed 2012-era restore client, and no lockfile in any edition

| | |
| --- | --- |
| **F-08-18** | **Restore depends on an executable committed to source control, and no lockfile records a content hash or enforces the resolved set at restore time** |
| **Severity** | **Medium** |
| **Editions** | MVC 4 commits the client; the missing lockfile is all three |

```bash
stat -c '%s' src/MVC4/MvcMusicStore/.nuget/NuGet.exe      # -> 630784
git ls-files 'packages.lock.json' | wc -l                 # -> 0
```

A NuGet client of version `2.0.30828.5` is tracked at [src/MVC4/MvcMusicStore/.nuget/NuGet.exe:630,784 bytes] — an executable, so its locator is the byte size that the `stat` command above reads and not a line number the artifact cannot support, which is the form section 1.4 requires of evidence with no line to point at; the version fields come from the PowerShell probe deliverable 02 §5.1 carries. It is what MVC 4's MSBuild-integrated restore invokes — a restore mechanism deprecated since NuGet 3. No edition has a `packages.lock.json`.

**What the missing lockfile costs here is not floating versions, and stating it as such would be wrong about the format.** `packages.config` is a flat list of every installed package **including transitive ones**, each at one exact version, and restore installs the list rather than re-resolving a graph — deliverable 02 §7.1 establishes this and §7.2 corroborates it from the two committed payloads. So the enumerated package set is exact. What no entry carries is a **content hash**, so a package with the expected id and version but different content restores without complaint; what no entry binds is the **source** it resolves from, which §6 shows is configured nowhere; and what no restore performs is a **locked-mode comparison** against a previously recorded resolution, because nothing records one. Deliverable 02 owns the inventory detail: §5.1 for the client and its properties, §7.1 for the absent lockfile and central version management, and §6 for the unconfigured source that makes the effective source set unknowable from the repository.

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

**Deliverable 10 owns the build outcomes, the diagnosis and the workarounds, and this register does not restate them.** What belongs here is that the debt exists, is platform-independent, and lives in committed files: an unconditional import [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:360] and a set of 24 `HintPath` package paths [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:66-157] in the MVC 4 project file, and the stale solution at [src/MVC4/MvcMusicStore/MvcMusicStore.sln:4], whose project declaration resolves one directory too deep. The equivalent declaration in the correct solution [src/MVC4/MvcMusicStore.sln:4] carries the identical relative path from one level up, which is exactly why the deeper file is the stale one. Section 10.2 records the solution-count hygiene aspect.

| | |
| --- | --- |
| **Remediation** | Not repaired here (section 1.3), and not carried forward: the target has one solution and SDK-style projects, so the defects are retired rather than fixed. Until then the established fact is the negative one: the edition cannot be built from its committed configuration, so any build of it would require some form of command-line compensation. Deliverable [10 §6.3](10-build-and-deployment-requirements.md) owns that ground and states its limit — it identifies candidate host-side compensations, records that no MVC 4 build under the prescribed toolchain was performed, and therefore leaves their sufficiency unestablished. This register names no invocation and claims none works, because none has been observed. |
| **Owner** | Deliverable 10 (build ownership); the repository owner if the legacy edition is ever to be built without compensation |

---

## 9. Dead scaffolding

Template scaffolding that executes, or ships, while serving nothing. Each entry has a cost: startup work, deployed surface, or a reader's time spent understanding a capability that does not exist.

### 9.1 F-08-20 — Area registration with no areas (all three editions)

| | |
| --- | --- |
| **F-08-20** | **`AreaRegistration.RegisterAllAreas()` runs at every application start and discovers nothing** |
| **Severity** | **Low** — a reflection scan at startup and a misleading signal about the application's structure |
| **Editions** | **All three.** Each calls it from its own `Application_Start`, and none has an `Areas` folder, so the scan discovers nothing in any edition |

`AreaRegistration.RegisterAllAreas();` is the first statement of `Application_Start` in MVC 5 [src/MVC5/MvcMusicStore/Global.asax.cs:15] and in MVC 4 [src/MVC4/MvcMusicStore/Global.asax.cs:19], and MVC 3 calls it too [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Global.asax.cs:36] — from the application class itself rather than from an `App_Start` file, because MVC 3 has no `App_Start` folder (deliverable 01 §3.6, which owns MVC 3's composition shape). The absence of the composition folder is a difference in *where* the call is written, not in whether it happens. No edition has an `Areas` folder:

```bash
git grep -n 'RegisterAllAreas' -- 'src/*' | grep -v /packages/
# -> src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Global.asax.cs:36
#    src/MVC4/MvcMusicStore/Global.asax.cs:19
#    src/MVC5/MvcMusicStore/Global.asax.cs:15
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

Deliverable 02 records the dependency side as F-02-07: four Web API packages pinned and referenced, one of them a metapackage with no assembly, serving this route. Deliverable 01 §9.3 marks the HTTP API capability **Unreachable** for MVC 4 rather than implemented — the surface exists, because the route template is mapped at [src/MVC4/MvcMusicStore/App_Start/WebApiConfig.cs:12-16], and it cannot be exercised, because no `ApiController` implements it. That is the distinction that keeps a mapped route from being read as a delivered feature, and it is not the same as **Absent**, which is the mark 01 gives the same capability in MVC 3 and MVC 5, where no such route is mapped at all.

| | |
| --- | --- |
| **Remediation** | Nothing to migrate: the route and its four packages are dropped. If an HTTP API is ever wanted, it is net-new work with its own approval. |
| **Owner** | The port (as a non-migration); deliverable 04 for the package disposition |

### 9.3 F-08-22 — Scaffolded, disabled external-login provider registrations and their dormant packages (MVC 5 and MVC 4)

| | |
| --- | --- |
| **F-08-22** | **Every external sign-in registration is inactive because it is commented out — three of the four also carry an empty credential pair, and Google's carries no argument at all and is complete as written — while the packages that would serve them ship and deploy** |
| **Severity** | **Low** as dead code; it becomes **High** the moment anyone uncomments a registration. For the three that take a credential pair, that also means supplying and protecting real credentials; for Google it means nothing further, because its call needs no argument — removing the comment markers is the whole of the change |
| **Editions** | MVC 5 and MVC 4 both carry the disabled surface, in different stacks. MVC 3 has no external-login surface at all. |
| **Remediation** | **Decided, not open**, and scoped to the **provider registrations and the dormant provider packages**: both are **removed**, and no `Microsoft.AspNetCore.Authentication.*` provider package replaces them. Registrations must not be carried forward commented out, which would reproduce this debt in the target. Re-enabling external sign-in later is a separately approved addition whose provider packages are pinned at the time it is approved, with credentials supplied from a secret store rather than from source. This removal is one half of a **split**: the linked-login list and its `Disassociate` removal action are **retained** and become a view component, which is outside this remediation. [05 §8.3](05-aspnet-core-migration-approach.md) owns the split; [12 §7.3](12-migration-blockers.md#73-the-three-decisions-this-document-defers-to-05--and-what-05-has-settled) names the removed action paths individually with their locators. This register cites both rather than re-deciding either. |
| **Owner** | [05](05-aspnet-core-migration-approach.md) for the split; the port to execute it, with security if external sign-in is ever enabled |

**MVC 5.** `ConfigureAuth` is 37 physical lines, of which 14 are comment lines, and the four commented-out provider registrations — Microsoft Account, Twitter, Facebook, Google — span [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:22-35]. **Their prerequisites are not uniform, so the credential clause holds for three of the four and not for all of them:** `UseMicrosoftAccountAuthentication` [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:23-25] with `clientId` and `clientSecret`, `UseTwitterAuthentication` [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:27-29] with `consumerKey` and `consumerSecret`, and `UseFacebookAuthentication` [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:31-33] with `appId` and `appSecret` are each written with an empty-string credential pair — while `app.UseGoogleAuthentication();` [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:35] **takes no argument list at all**. What makes the whole surface inactive is therefore the comment markers rather than an absent secret; deliverable [09 §6.11](09-security-assessment.md#611-scaffolded-but-disabled-external-login-ships-as-deployed-attack-surface) owns that current-state attribution and its per-provider prerequisites, and this register cites it rather than restating it. Only two calls are live: cookie authentication [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:14-18] and the external sign-in cookie [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:20], the latter existing solely to support the providers that are disabled.

**MVC 4.** The same shape in a different stack, including the same asymmetry: `RegisterAuth` is 32 physical lines with 12 comment lines, and the four commented `OAuthWebSecurity.Register*Client` calls span [src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs:17-29] — `RegisterMicrosoftClient` [src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs:17-19], `RegisterTwitterClient` [src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs:21-23] and `RegisterFacebookClient` [src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs:25-27] with empty credential pairs, and `OAuthWebSecurity.RegisterGoogleClient();` [src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs:29] with no arguments at all.

```bash
for f in src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs; do
  printf '%s physical=%s comment_lines=%s\n' "$f" "$(wc -l < "$f")" "$(grep -c '^[[:space:]]*//' "$f")"
done                                    # -> 37/14 and 32/12
```

The dependency cost is deliverable 02's, and it is not symmetric: MVC 5 ships **four** dormant `Microsoft.Owin.Security.*` provider packages (§3.1.2, F-02-03) and MVC 4 ships **six** DotNetOpenAuth packages plus the WebPages OAuth surface (§3.2.3, F-02-08). One package is easy to miscount here: `Microsoft.Owin.Security.OAuth` is **OAuth infrastructure, not a fifth provider** — deliverable 02 §3.1.2 states it explicitly.

**Read together, those two facts make this entry more immediate rather than less, which is why the credential correction sharpens the debt instead of softening it.** The provider assemblies are already restored and copied to the output directory of both editions, and in both editions the Google registration is complete as written — so for that one provider the entire distance between shipped-and-disabled and shipped-and-live is a two-character deletion, with nothing to obtain, provision or store on the way. The three registrations that take a credential pair are at least gated by somebody having to acquire and protect a secret first. That asymmetry is the reason the remediation row above removes the registrations outright rather than carrying them forward commented out: for one of the four, the comment marker *is* the control.

A related consequence sits in the views rather than the startup files: MVC 5's account management views render an external-login removal list [src/MVC5/MvcMusicStore/Views/Account/Manage.cshtml:22] whose provider-enumeration partial has **no provider to enumerate**, because every registration is commented out [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:22-35] — so the sign-in-with-a-provider path is unreachable as shipped, and `_ExternalLoginsListPartial.cshtml`'s empty-provider branch [src/MVC5/MvcMusicStore/Views/Account/_ExternalLoginsListPartial.cshtml:7-13] is the branch that renders today. The same holds in MVC 4 [src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs:17-29].

**That is the whole of what the evidence supports, and the stronger claim is deliberately not made.** "No provider is registered" does **not** establish that no persisted external-login association exists or can be removed: an association row can predate a registration being commented out, and the removal list is populated from the user's stored logins rather than from the registered providers, so a row that exists would still render and still be removable. Establishing that the removal list is genuinely always empty would require authoritative row evidence from the committed Identity database — which is tracked in this repository (F-08-11) but whose contents this assessment does not read, for the reason deliverable 12 F-12-21 states: probing a database binary is evidence, not proof, and the authoritative answer is a query against the attached database. This register therefore records the registration state, which is a repository fact, and not the row state, which is not. Deliverable 05 owns whether that surface is ported or removed, **and has decided it**: the split this entry's remediation row records, with the sign-in and linking half removed and the linked-login management half retained.

---

## 10. Repository hygiene

All entries in this section are **Low** severity with **no migration impact**. They are recorded because they are quantified, because they shape a newcomer's first impression of the repository, and because one of them — the root `.gitignore` — is the evidence that makes several of the higher-severity entries above *debt* rather than a deliberate choice.

### 10.1 The primary evidence: root `.gitignore`, line by line

A tracked file that the repository's own rules exclude was added before the rule existed, and `.gitignore` cannot untrack it. That asymmetry is what distinguishes debt from an intentional decision, so the root `.gitignore` — 34 pattern lines in all [.gitignore:1-34] — is cited line by line as the primary evidence for the whole gitignored-yet-tracked class:

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
| [src/MVC5/MvcMusicStore.sln:6] | Current, MVC 5 — its project declaration resolves |
| [src/MVC4/MvcMusicStore.sln:4] | Current, MVC 4 — its project declaration resolves |
| [src/MVC3/MvcMusicStore-Completed/MvcMusicStore.sln:4] | Current, MVC 3 — its project declaration resolves |
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

[src/MVC4/MvcMusicStore-Create.sql:1-629] and [src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql:1-629], 629 lines each, `cmp`-identical. Two copies of an unrunnable script are two chances to adopt the wrong baseline.

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

**Tracked against [.gitignore:33] — and the exclusion, unlike the count, is host-dependent.** `Packages/` is the only rule that matches these paths: its sole separator is trailing, so it matches a directory of that name at any depth, and it reaches the lowercase `packages` directories only because `core.ignorecase` is `true` on this checkout. `packages/*` [.gitignore:15] does **not** cover them: an interior separator anchors a pattern to the directory holding the `.gitignore`, so that rule reaches a root-level `packages/` and nothing nested. On a case-sensitive filesystem neither rule matches, so *gitignored-yet-tracked* is the accurate description of these 215 files on a case-insensitive host and *simply tracked* is the accurate description elsewhere. Section 10.7 carries the probe, the `core.ignorecase` experiment and the second finding this dependence belongs to (F-08-28), and deliverable [04 §A.6](04-dotnet8-migration-strategy.md) states the same pattern analysis. The debt is unchanged either way: 215 tracked files of restored third-party payload, 32 of them binaries, is the finding — the ignore rules only narrow *why* the tracking is anomalous. Deliverable 02 §7.2 owns the inventory (F-02-20) and records the asymmetry that matters more than the count: MVC 5 commits **no** packages, so the two editions cannot be prepared the same way — a fact deliverable 10 carries as a build requirement.

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

[src/MVC3/MVC Music Store - Tutorial - v3.0.pdf:4,993,295 bytes] is the single largest non-database file in the repository, more than three times the size of the next largest. It is a document binary, so its locator is that byte size, established by the two commands above — the form section 1.4 requires of evidence with no line to point at. The licensing distinction is the part worth recording, because it survives any decision about the file: the code is under the Microsoft Public License while the tutorial document is under Creative Commons Attribution 3.0 [src/MVC3/readme.txt:5]. Anything derived from the document therefore carries an attribution obligation that the code does not. This document treats the PDF as repository weight and licensing evidence only; it was not read for requirements.

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

**First, the correct probe.** `git check-ignore -v <path>` exits **1 with no output for every tracked file** in this repository — verified on `NuGet.exe`, both `App_Data` catalogs, both `.csproj.user` files, the `.suo` and a `packages/` payload — because `check-ignore` consults the index before the ignore rules. The exit code therefore says nothing about whether a rule matches. `--no-index` is the probe that answers the question, and with it every gitignored-yet-tracked claim in section 10.1 verifies — each against the rule the probe names, which for the two `packages/` trees is `Packages/` [.gitignore:33] rather than the root-anchored `packages/*` [.gitignore:15], and with the case dependency that finding two attaches to it:

```bash
git check-ignore --no-index -v src/MVC4/MvcMusicStore/.nuget/NuGet.exe
# -> .gitignore:28:nuget.exe        src/MVC4/MvcMusicStore/.nuget/NuGet.exe
git check-ignore --no-index -v src/MVC5/MvcMusicStore/App_Data/MvcMusicStore.mdf
# -> .gitignore:32:App_Data/        src/MVC5/MvcMusicStore/App_Data/MvcMusicStore.mdf
git check-ignore --no-index -v src/MVC4/MvcMusicStore/MvcMusicStore.v11.suo
# -> .gitignore:8:*.suo             src/MVC4/MvcMusicStore/MvcMusicStore.v11.suo
git check-ignore --no-index -v src/MVC4/MvcMusicStore/packages/repositories.config
# -> .gitignore:33:Packages/        src/MVC4/MvcMusicStore/packages/repositories.config
git check-ignore --no-index -v src/MVC3/MvcMusicStore-Assets/Data/ASPNETDB.MDF
# -> (exit 1, no output: no rule matches this path at all)
```

**Finding one — four of the fourteen database binaries are not ignored by any rule.** The last probe above is the interesting one. `.gitignore:32` is `App_Data/`, and MVC 3's four binaries live under `Data/`, which no pattern covers. So F-08-11 splits cleanly: **ten** files are gitignored-yet-tracked, and **four** are simply tracked, with no rule in the current `.gitignore` matching their path. A remediation that assumes one uniform cause would miss the second group.

**Both statements are extended from the checkout to the tracked history by command, because history is not something a snapshot can establish.** `.gitignore` has exactly two revisions in this clone, `App_Data/` is the only `Data`-bearing rule either of them ever introduced, and the binaries entered the index before that rule existed:

```bash
git log --all -p --pretty=format:'%h' -- .gitignore | grep -E '^\+[^+]' | sort -u | grep -i data
# -> +App_Data/   the only Data-bearing rule in any revision; nothing matches src/MVC3/.../Data/
git merge-base --is-ancestor 1a374e6 80d24f4 && echo 'binaries precede the first App_Data/ rule'
# -> binaries precede the first App_Data/ rule
#    1a374e6 'Added MVC 4 and MVC 5 versions' 2022-11-04 16:31; 80d24f4, the first .gitignore
#    revision carrying App_Data/ at its line 32, 2022-11-04 17:18
```

**Finding two — two rules stop matching on a case-sensitive filesystem, and one of them carries a whole classification with it.** The first is `nuget.exe`: [.gitignore:28] spells the pattern in lowercase while the tracked path is `NuGet.exe` [src/MVC4/MvcMusicStore/.nuget/NuGet.exe:630,784 bytes] — an executable, so its locator is the byte size, per section 1.4, and the tracked spelling that the rule has to match is read by command rather than by line: `git ls-files 'src/MVC4/MvcMusicStore/.nuget/*'` → `src/MVC4/MvcMusicStore/.nuget/NuGet.Config`, `src/MVC4/MvcMusicStore/.nuget/NuGet.exe`, `src/MVC4/MvcMusicStore/.nuget/NuGet.targets`. The probes below print the same spelling inside `git check-ignore` output. Whether the pattern matches depends on `core.ignorecase`, which is `true` on this Windows checkout. Verified by a controlled experiment in a throwaway repository created **outside** the checkout with the same `.gitignore` and deleted afterwards — the one non-read-only command in this document, and it touched no repository file:

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

**The second rule is `Packages/`, and there the case dependence decides how 215 tracked files are classified.** [.gitignore:33] is spelled with a capital while both committed directories are lowercase, so it reaches them only under `core.ignorecase = true`; `packages/*` [.gitignore:15] never reaches them at all, because its interior separator anchors it to the directory holding the `.gitignore` — the repository root. Both halves are established inside the checkout and read-only, because `-c` supplies the setting for one invocation and writes nothing:

```bash
git -c core.ignorecase=false check-ignore --no-index -v \
  src/MVC4/MvcMusicStore/packages/repositories.config \
  src/MVC3/MvcMusicStore-Completed/packages/repositories.config
# -> no output (exit 1: on a case-sensitive host NEITHER tree is matched by any rule)
git -c core.ignorecase=false check-ignore --no-index -v packages/x
# -> .gitignore:15:packages/*  packages/x   (the root-level path the anchored rule does cover)
```

So the 215 files of F-08-25 are gitignored-yet-tracked on this checkout and simply tracked on a case-sensitive one. That narrows the reason they are anomalous without reducing the debt, which is the tracking of restored third-party payload; section 10.4 states it that way, and deliverable [04 §A.6](04-dotnet8-migration-strategy.md) records the same two-rule analysis for the payload the assessment's own restores wrote.

| | |
| --- | --- |
| **F-08-28** | **The hygiene configuration is itself host-dependent: the effect of two ignore rules changes with filesystem case sensitivity, and one class of committed binary is covered by no rule** |
| **Severity** | **Low** in its own right. Its value is corroborative. |
| **Editions** | Repository-wide — the root `.gitignore` and the paths it does and does not cover |
| **Evidence** | The probes and experiments above |
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
| F-08-02 | Two files register the EF initializer (duplicated configuration) | Code | Low | 5 for the duplication; the registration itself is in 3, 4, 5 — §5.1 | Migration workstream |
| F-08-03 | Uncached nested aggregate and cart read on every page | Code | High | 3, 4, 5 | Performance |
| F-08-04 | Unbounded result sets in two list actions | Code | Medium | 3, 4, 5 | The port |
| F-08-05 | 4 `Single` calls on unvalidated input; 3 unguarded `Find` results; 1 unreachable guard | Code | Medium | 3, 4, 5 (guard distribution differs — §5.4) | The port; 05 for target contracts |
| F-08-06 | 5 unvalidated state-changing POSTs; a state-changing `GET`; MVC 3 validates nothing | Code | High | 3, 4, 5 | Security |
| F-08-07 | Plaintext administrator credential, consumed at startup | Code | High | 4, 5 | Security |
| F-08-08 | Bare `catch` around the order write | Code | High | 3, 4, 5 — §5.7; MVC 3's split-save transaction shape makes its instance worse, not absent | The port |
| F-08-09 | Hand-constructed contexts with `Dispose(bool)` overrides | Code | Medium | 3, 4, 5 | The port |
| F-08-10 | `DropCreateDatabaseIfModelChanges` over orders and PII | Data | **Critical** | 3, 4, 5 | Data workstream |
| F-08-11 | 14 database binaries, 43,376,640 bytes — two declared credential stores plus one unreferenced, credential-shaped tutorial database (§6.2, §6.2.1) | Data | High | 3, 4, 5 | Repository owner; security |
| F-08-12 | Schema scripts not runnable as written; none for MVC 5 | Data | Medium | 4 (3 assets; 5 none) | Data workstream |
| F-08-13 | No logging, tracing, metrics or health endpoint | Operational | **Critical** | 3, 4, 5 | Operations and platform |
| F-08-14 | No CI, no deployment automation, no publish artifact | Operational | High | repo-wide | Operations and platform |
| F-08-15 | No test of any kind | Operational | **Critical** | repo-wide | The port; 07 for risk |
| F-08-16 | View compilation disabled; 29 views uncheckable | Build | Medium | 3, 4, 5 | The port |
| F-08-17 | Warning level 4 set, enforcement absent | Build | Medium | 3, 4, 5 | The port; operations |
| F-08-18 | Committed 2012-era restore client; no lockfile | Build | Medium | 4; all for the lockfile | The port; operations |
| F-08-19 | MVC 4 build configuration broken; fourth solution stale | Build | High | 4 | Deliverable 10; repository owner |
| F-08-20 | Area registration with no areas | Dead scaffolding | Low | 3, 4, 5 — §9.1; MVC 3 calls it from the application class rather than an `App_Start` file | Migration workstream |
| F-08-21 | Mapped HTTP API route, zero `ApiController` | Dead scaffolding | Low | 4 | The port; 04 |
| F-08-22 | External-login surface scaffolded and disabled | Dead scaffolding | Low | 4, 5 | The port; security; 05 for the split |
| F-08-23 | Four solutions for three projects, one stale | Hygiene | Low | 4 | Repository owner |
| F-08-24 | Schema script committed twice | Hygiene | Low | 4 | Repository owner |
| F-08-25 | 215 committed `packages/` files, 32 binaries | Hygiene | Low | 3, 4 | Repository owner |
| F-08-26 | 4,993,295-byte tutorial PDF with distinct licensing | Hygiene | Low | 3 | Repository owner |
| F-08-27 | Three IDE user-state files tracked | Hygiene | Low | 4, 5 | Repository owner |
| F-08-28 | Ignore rules host-dependent; one binary class uncovered | Hygiene | Low | repo-wide | Repository owner |

**Distribution by severity: 3 Critical, 8 High, 7 Medium, 10 Low — 28 rows.** The three Critical entries share a property worth naming: none of them is a bug in the ordinary sense. Two are absences — no observability, no tests — and one is a configured behaviour working exactly as designed. A defect-hunting review would have found none of the three, which is why an assessment that inventories absence as well as presence is the only kind that reaches them.

**Every distribution in this section and in section 11.1 is regenerated from the table above by the single command in section 14, and each of the four is a partition: severity, category, primary owner and edition class each sum to the register's 28 rows.** None is maintained by hand, because a roll-up computed once and then edited around is exactly how a register acquires a total its own rows do not support.

**Distribution by category.**

| Category | Entries | The entries |
| --- | --- | --- |
| Code | **8** | F-08-02, 03, 04, 05, 06, 07, 08, 09 |
| Hygiene | **6** | F-08-23, 24, 25, 26, 27, 28 |
| Build | **4** | F-08-16, 17, 18, 19 |
| Data | **3** | F-08-10, 11, 12 |
| Dead scaffolding | **3** | F-08-20, 21, 22 |
| Operational | **3** | F-08-13, 14, 15 |
| Duplication | **1** | F-08-01 |
| **Total** | **28** | — |

**Distribution by edition class, with incidence reported separately because it is not a partition.** The class is each row's **leading edition assertion** — the scope stated before any parenthesis, semicolon or dash qualifier — so the classes partition the register and sum to 28. Six rows carry a qualifier after that assertion (F-08-02, 05, 08, 12, 18, 20), and the qualifier stays in the row rather than moving the class, so the partition below is read together with those rows and not instead of them.

| Edition class | Entries | The entries |
| --- | --- | --- |
| All three | **13** | F-08-01, 03, 04, 05, 06, 08, 09, 10, 11, 13, 16, 17, 20 |
| MVC 4 only | **6** | F-08-12, 18, 19, 21, 23, 24 |
| MVC 4 and MVC 5 | **3** | F-08-07, 22, 27 |
| Repository-wide | **3** | F-08-14, 15, 28 |
| MVC 3 and MVC 4 | **1** | F-08-25 |
| MVC 5 only | **1** | F-08-02 |
| MVC 3 only | **1** | F-08-26 |
| **Total** | **28** | — |

Counting a row once for every edition its class names gives **MVC 3 in 15 entries, MVC 4 in 23 and MVC 5 in 17**, alongside the 3 repository-wide rows that name no edition. Those three figures sum to more than 28 by construction, so they are incidence and never a total: the only claim they support is relative exposure, and MVC 4's being the highest follows from its carrying a broken build configuration, a stale solution, a duplicated schema script and a dead API route on top of everything the three editions share.

---

### 11.1 Owner roll-up, with its counting rule stated

**The counting rule, because the answer depends on it.** A finding is counted **once for every owner named in its row of the table above**, so a finding with two owners appears in two rows of the roll-up and the per-owner counts sum to more than 28. The `Owner` column of that table is the canonical input: where an entry's own `Owner` line names an additional *reviewing* party that the table row compresses — §5.7 names security as reviewing disclosure on F-08-08 — the review is not counted as an owner assignment. Deliverables named as co-owners are counted separately from the seven roles of section 1.5, because a deliverable owns a decision rather than a remediation.

| Owner | Entries | The entries | Severity mix |
| --- | --- | --- | --- |
| The port | **11** | F-08-01, 04, 05, 08, 09, 15, 16, 17, 18, 21, 22 | 1 Critical, 2 High, 6 Medium, 2 Low |
| The repository owner | **8** | F-08-11, 19, 23, 24, 25, 26, 27, 28 | **2 High** (F-08-11, F-08-19), 6 Low |
| Security | **4** | F-08-06, 07, 11, 22 | 3 High, 1 Low |
| Operations and platform | **4** | F-08-13, 14, 17, 18 | 1 Critical, 1 High, 2 Medium |
| The data workstream | **2** | F-08-10, 12 | 1 Critical, 1 Medium |
| The migration workstream | **2** | F-08-02, 20 | 2 Low |
| Performance | **1** | F-08-03 | 1 High |
| **Total role assignments** | **32** | 28 findings, of which **4** name two roles each (F-08-11, F-08-17, F-08-18, F-08-22) — 28 + 4 = 32 | — |
| Deliverables named as co-owners | **6** | 07 (F-08-01, F-08-15), 05 (F-08-05, F-08-22), 10 (F-08-19), 04 (F-08-21) | — |

Two properties of these counts are stated explicitly, because a roll-up that summarizes severity loosely is worse than no roll-up at all:

- **The repository owner's set is not uniformly Low.** Six of the eight are Low hygiene items, but **F-08-11 (14 committed database binaries — two credential-bearing application stores plus one unreferenced credential-shaped tutorial database) and F-08-19 (MVC 4's broken build configuration and the stale solution) are both High**, and F-08-19 is co-owned with deliverable 10. Summarizing this owner's set as low-consequence would invite deferring the two items in it that are not.
- **The port carries the largest set, at 11, and it spans all four severities** — one Critical (F-08-15, no test of any kind), two High, six Medium, two Low. The Critical one is the entry the pre-port sequencing in deliverable 03 turns on.

Nothing in this roll-up changes an entry's severity, owner or identifier: it is the same 28 findings, distributed 3 Critical, 8 High, 7 Medium, 10 Low.

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
| Views to port | **29** Razor files in MVC 5, dividing **3 component + 5 other special + 21 mechanical = 29**; separately, **6** of the 29 are source views naming a removed API or type | Count: deliverable 01 §2.5. Both divisions: deliverable 05 §8.3, §8.4 | Cite 01 for the 29 and 05 for either division, not this register. **Do not substitute one unit for the other** — the 5 is per-line work, the 6 is removed-API surface, and section 8.1 states why they differ |
| Static assets to relocate | **27** in MVC 5's four asset groups | Deliverable 01 §2.3 | Cite 01 |
| Unvalidated state-changing POSTs | **5** in MVC 5, **5** in MVC 4, **8** in MVC 3 | Census, section 5.5 | Scope of the anti-forgery work per edition |
| Manual construction sites | **10** in MVC 5 | Deliverable 01 §5.4 | Scope of the injection work |
| Database binaries | **14** files, **43,376,640** bytes | Section 6.2 | Repository-hygiene decision, not migration effort |
| Committed package files | **215** | Section 10.4 | Same |
| Tests to write from zero | **0** exist | Section 7.3 | The pre-port suite is entirely net-new |

### 12.2 Quantities that must not be used as effort inputs

- **422 and 426 physical lines** for `AccountController.cs` (section 3.3) — duplication metric. The sizing figure is 382.
- **All diff-line counts** in section 3 — 397, 414, 668, 272, the 7-to-35 range, the 2-line `Album.cs` delta. They measure divergence between two existing files, not work to be done.
- **826 LF, 827 content lines** for the seed (section 6.1) — the physical-line metric, per deliverable 01 §2.4. The sizing figure for the same file is 820 non-blank lines, and the two must not be added or substituted for one another.
- **Severity ratings.** Severity is consequence, not cost. A Low-severity entry can be expensive (F-08-25's 215 committed package files are Low, and remediating them is a repository-history decision) and a Critical one can be cheap to decide (F-08-10 is a design choice).
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
| Layout query fan-out per edition; the four cache-invalidation paths | F-08-03, §5.2.1 | 05 (view components; and the acceptance criteria — one database read on a cold miss, zero on a warm hit); **05** §8.2 (the caching mechanism — a versioned key over the shared SQL-backed distributed cache, absolute 60-second expiration, per-key single-flight and a three-case degraded contract, so a stamp rotation converges every replica at once and the expiry covers only the order trigger); **06** §6.4 (the sizing consequence). Both cited element by element in §5.2.2 |
| Anti-forgery coverage today; the state-changing `GET` | F-08-06 | 05 (policy, verb change, token transport); 09 (posture) |
| Plaintext credential and the `async void` provisioning | F-08-07 | 09 (security analysis); 05 (provisioning mechanism) |
| Bare `catch` and absent logging together | F-08-08, F-08-13 | 09 (disclosure); 06 (telemetry); 05 (pipeline) |
| Duplicate `Genre.Name` and what the browse path must do about it | F-08-05, §5.4.1 | **05** — the duplicate-`Genre` browse contract in full: non-lossy unconditionally, gated on the schema extraction, with the duplicates branch a conditional approved delta whose approval owners (Product and Data) and coverage rows 05 records. **This register** records the measured source behaviour, the debt and the rejection of first-match-wins, and adds no delta or coverage row |
| Hand-constructed contexts and disposal overrides | F-08-09 | 05 (injection and lifetimes) |
| Destructive initializer; no schema script for MVC 5 | F-08-10, F-08-12 | 05 and 06 (schema lifecycle, deployment-time application); 12 (differing defaults). **Inbound:** F-08-10 (§6.1) is the closure for 09's **F-09-31** |
| 14 committed binaries: **two credential-bearing application stores plus one unreferenced credential-shaped tutorial database**, and that database's classification | F-08-11, §6.2.1 | 09 (credential exposure); **10 §10.1–§10.2** (per-edition database topology, and §13.2 item 2 keeping MVC 3's host-inherited store **unverified**); repository owner (history). **Inbound:** F-08-11 (§6.2) is the closure for 09's **F-09-34** |
| No observability; no CI; no tests | F-08-13, F-08-14, F-08-15 | 03 (workstreams and gates); 06 (telemetry); 07 (risk) |
| Disabled view compilation; absent enforcement | F-08-16, F-08-17 | 10 (build requirements); 04 (target project properties) |
| Committed restore client; no lockfile; unpinned source | F-08-18 | 02 §5–§7 (inventory); 04 (target pins and lockfile); 10 (build) |
| MVC 4 configuration defects; stale solution | F-08-19 | **10** (build outcomes and workarounds) |
| Dormant provider and Web API packages | F-08-21, F-08-22 | 02 §3.1.2 and §3.2.3 (inventory); 04 (disposition) |
| Path casing in the ignore rules and in the bundle registration | F-08-28 | 06 and 11 (Linux hosting and the casing audit) |
| Repository weight and hygiene | F-08-23 – F-08-27 | Repository owner; none blocks the migration |

**Inputs consumed by this register:** deliverable 01 §2.3–§2.5 (counts and code volume), §3.4 (double registration), §5.3–§5.4 (child actions and construction sites), §6.4 (schema lifecycle and seed), §7 (the two unit-of-work models), §9.3 (capability gaps), §10 (cross-edition comparison); deliverable 02 §3.1.2, §3.2.2, §3.2.3 (dormant and unused packages), §5.1 (the committed client), §6 (the unconfigured source), §7.1–§7.2 (lockfile and committed payloads), §8.1–§8.3 (the aged-pin class with its exact pins, why no advisory identifier is asserted and the route a reader runs instead, and the scan the implementation phase owes).

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

# --- §5.2.1  Who writes Genres: the seed only, no controller in any edition -------
git grep -nE 'Genres\.(Add|AddRange|Remove|RemoveRange|Attach)' -- '*.cs'
# -> src/MVC4/MvcMusicStore/Models/SampleData.cs:822:  genres.ForEach(s => context.Genres.Add(s));
#    src/MVC5/MvcMusicStore/Models/SampleData.cs:822:  genres.ForEach(s => context.Genres.Add(s));
# Two lines, both the seed; no Controllers/ path appears. MVC 3's seed writes no
# Genres set at all -- it attaches Genre instances through the Album graph instead.

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
git ls-files '*.cs' '*.config' '*.cshtml' '*.csproj' '*.sln' '*.sql' '*.txt' '*.md' \
  | grep -v /packages/ | grep -v '^docs/' | xargs grep -il 'ASPNETDB' | wc -l    # -> 0
#    the tutorial credential-shaped database is referenced by no tracked application artifact

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
# customErrors: three counting units, each with its own command
git ls-files -- '*Web.Debug.config' '*Web.Release.config' | grep -v '/packages/' \
  | xargs grep -o 'customErrors'  | wc -l     # -> 24  occurrences of the word (4 per file)
git ls-files -- '*Web.Debug.config' '*Web.Release.config' | grep -v '/packages/' \
  | xargs grep -o '<customErrors' | wc -l     # -> 12  matches of the literal (2 per file)
git ls-files -- '*Web.Debug.config' '*Web.Release.config' | grep -v '/packages/' | while read -r f; do
  awk -v F="$f" '/<!--/{i=1} /<customErrors/{if(i)a++;else b++} /-->/{i=0}
                 END{printf "%s inside=%d outside=%d\n", F, a+0, b+0}' "$f"
done                     # -> all six files inside=2 outside=0: zero live elements
sed -n '19,29p' src/MVC5/MvcMusicStore/Web.Release.config
# -> the representative comment block: <!-- at :19, example element at :25, --> at :29
git ls-files | grep -c '^\.github/'                                              # -> 0
git ls-files | grep -ciE 'azure-pipelines|jenkinsfile|appveyor|\.travis'          # -> 0
git ls-files | grep -ci 'pubxml'                                                 # -> 0
git log --all --diff-filter=A --name-only --pretty=format: -- '*.pubxml' '*.pubxml.user' | sort -u
git log --all --diff-filter=A --name-only --pretty=format: -- '*PublishProfiles/*' 'build/*' | sort -u
# -> both produce no output: no such path was added on any ref in this clone
git ls-files | grep -ciE '\.(log|binlog)$'                                       # -> 0  no warning record
git ls-files | grep -ciE 'dockerfile|docker-compose|\.ya?ml$'                     # -> 0
git ls-files | grep -v /packages/ | grep -ciE '\.(ps1|sh|cmd|bat)$'               # -> 0
git ls-files '*.cs' '*.csproj' | grep -v /packages/ \
  | xargs grep -lE 'TestClass|\[Fact\]|xunit|NUnit|Microsoft\.VisualStudio\.TestTools' | wc -l   # -> 0

# --- §8  Build debt ---------------------------------------------------------------
grep -n 'MvcBuildViews' src/MVC5/MvcMusicStore/MvcMusicStore.csproj               # -> :17 false, :274 gated
git ls-files '*.csproj' | grep -v /packages/ | xargs grep -n 'WarningLevel'       # -> 4 in all three
git ls-files '*.csproj' | grep -v /packages/ | xargs grep -c 'TreatWarningsAsErrors'   # -> 0
git ls-files | grep -ciE '\.ruleset$|\.editorconfig$|Directory\.Build\.props'     # -> 0
git ls-files | grep -E '(^|/)packages\.lock\.json$'   # -> no output; matches at any depth, not just root
stat -c '%s' src/MVC4/MvcMusicStore/.nuget/NuGet.exe                             # -> 630784

# --- §9  Dead scaffolding ---------------------------------------------------------
git ls-files '*Global.asax.cs' | grep -v /packages/ | xargs grep -n 'AreaRegistration'
#  -> MVC3 .../Global.asax.cs:36, MVC4 .../Global.asax.cs:19, MVC5 .../Global.asax.cs:15
git ls-files | grep -ci '/Areas/'                                                # -> 0
git ls-files '*.cs' | grep -v /packages/ | xargs grep -c 'ApiController' | awk -F: '{s+=$NF} END {print s}'
                                                                                 # -> 0
for f in src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs; do
  printf '%s physical=%s comment_lines=%s\n' "$f" "$(wc -l < "$f")" "$(grep -c '^[[:space:]]*//' "$f")"
done                                                                             # -> 37/14 and 32/12
# the per-provider argument shape behind §9.3: the three credential-taking calls open a
# parameter list, Google's is closed on its own line and takes nothing
grep -nE '^[[:space:]]*//[[:space:]]*(app\.Use|OAuthWebSecurity\.Register)' \
  src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs \
  src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs
# -> MVC5 :23 UseMicrosoftAccountAuthentication(  :27 UseTwitterAuthentication(  :31 UseFacebookAuthentication(
#    MVC5 :35 UseGoogleAuthentication();
#    MVC4 :17 RegisterMicrosoftClient(  :21 RegisterTwitterClient(  :25 RegisterFacebookClient(
#    MVC4 :29 RegisterGoogleClient();

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
git ls-files 'src/MVC4/MvcMusicStore/.nuget/*'
# -> src/MVC4/MvcMusicStore/.nuget/NuGet.Config
#    src/MVC4/MvcMusicStore/.nuget/NuGet.exe        the tracked spelling the lowercase rule must match
#    src/MVC4/MvcMusicStore/.nuget/NuGet.targets
git check-ignore -v src/MVC4/MvcMusicStore/.nuget/NuGet.exe; echo "exit=$?"
# -> no output, exit=1 — TRACKED files always exit 1 without --no-index; says nothing about the rules
git check-ignore --no-index -v src/MVC4/MvcMusicStore/.nuget/NuGet.exe     # -> .gitignore:28:nuget.exe
git check-ignore --no-index -v src/MVC5/MvcMusicStore/App_Data/MvcMusicStore.mdf  # -> .gitignore:32:App_Data/
git check-ignore --no-index -v src/MVC4/MvcMusicStore/MvcMusicStore.v11.suo       # -> .gitignore:8:*.suo
git check-ignore --no-index -v src/MVC4/MvcMusicStore/packages/repositories.config # -> .gitignore:33:Packages/
git check-ignore --no-index -v src/MVC3/MvcMusicStore-Assets/Data/ASPNETDB.MDF    # -> exit 1: no rule matches
# the nested packages/ trees match Packages/ [.gitignore:33] only under core.ignorecase; -c writes nothing
git -c core.ignorecase=false check-ignore --no-index -v \
  src/MVC4/MvcMusicStore/packages/repositories.config \
  src/MVC3/MvcMusicStore-Completed/packages/repositories.config
# -> no output, exit 1: on a case-sensitive host no rule matches either tree
git -c core.ignorecase=false check-ignore --no-index -v packages/x  # -> .gitignore:15:packages/* (root only)
git log --all -p --pretty=format:'%h' -- .gitignore | grep -E '^\+[^+]' | sort -u | grep -i data
# -> +App_Data/  the only Data-bearing rule in any revision of the ignore file
git merge-base --is-ancestor 1a374e6 80d24f4 && echo 'binaries precede the first App_Data/ rule'
# -> true: 1a374e6 added the App_Data binaries at 2022-11-04 16:31; 80d24f4, the first .gitignore
#    revision carrying App_Data/, is 2022-11-04 17:18

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

# --- §1.5  One remediation and one owner per entry, counted ------------------------
grep -c '^| \*\*Remediation\*\* |' docs/modernization/08-technical-debt-register.md   # -> 28
grep -c '^| \*\*Owner\*\* |'       docs/modernization/08-technical-debt-register.md   # -> 28
grep -c '^| F-08-[0-9][0-9] |'     docs/modernization/08-technical-debt-register.md   # -> 28

# --- §11, §11.1  Every distribution, regenerated from the section 11 table ----------
# The register table is the only input. Field 2 is the identifier, 4 the category,
# 5 the severity, 6 the editions and 7 the owner; the primary owner is the text before
# the first ';' and the edition class is the leading edition assertion.
R=docs/modernization/08-technical-debt-register.md
awk -F'|' '/^\| F-08-[0-9][0-9] \|/ {
    n++; c=$4; s=$5; e=$6; o=$7
    gsub(/^ +| +$/,"",c); gsub(/[ *]/,"",s); gsub(/^ +| +$/,"",e); gsub(/^ +| +$/,"",o)
    split(o,p,";"); gsub(/^ +| +$/,"",p[1])
    match(e,/^(repo-wide|[345]( *, *[345])*)/); k=substr(e,1,RLENGTH); gsub(/ /,"",k)
    C[c]++; S[s]++; O[p[1]]++; E[k]++
  }
  END { printf "rows=%d\n", n
    for (x in S) printf "sev  %-9s %d\n", x, S[x]
    for (x in C) printf "cat  %-17s %d\n", x, C[x]
    for (x in O) printf "own  %-24s %d\n", x, O[x]
    for (x in E) printf "edn  %-9s %d\n", x, E[x] }' $R | sort
# -> rows=28
# -> sev  Critical 3 | High 8 | Medium 7 | Low 10                              (sum 28)
# -> cat  Code 8 | Hygiene 6 | Build 4 | Data 3 | Dead scaffolding 3 |
#         Operational 3 | Duplication 1                                        (sum 28)
# -> own  The port 11 | Repository owner 7 | Security 2 | Data workstream 2 |
#         Operations and platform 2 | Migration workstream 2 | Performance 1 |
#         Deliverable 10 1                                                     (sum 28)
# -> edn  3,4,5 13 | 4 6 | 4,5 3 | repo-wide 3 | 3,4 1 | 5 1 | 3 1            (sum 28)

# --- The constraint this work was held to (section 1.3's one range, short form) ----
# HEAD is the delivery commit defined in section 1.3 and ea2552d is the short form of
# the same immutable revision, so this is that range and not a second one.
git diff --name-status ea2552d..HEAD
# -> exactly 13 rows, every one an 'A' under docs/modernization/; no M and no D row
git diff --name-status ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD \
  | grep -c '^A'                # -> 13
git status --porcelain          # -> no output: no tracked file modified, none untracked
git status --porcelain --ignored # -> no output ONLY when nothing ignored is present. The
#    restored packages/ and the bin/ and obj/ trees the assessment's own builds wrote
#    were removed when it finished with them; these two commands equally report any
#    generated tree a later run leaves in the checkout, whoever produced it, so their
#    empty result is a pre-commit condition rather than a standing claim (02 §1.3).
git clean -ndX                  # -> no output (dry run) in that same cleared state
```

---

*Deliverable 08 of 13. Supporting assessment record: it consumes deliverables 01 and 02 and feeds deliverable 07's effort model and risk register. Twenty-eight entries, each with a severity, a remediation and an owner; every quantity re-derivable from the file or command cited beside it. No **tracked** repository file was modified in producing it — the ignored restore and build output the assessment did write was removed once it had finished with it, and section 1.3 records why only the first two of its four commands are a durable claim: the other two see ignored content, so they report whatever generated output is present when they run, whoever produced it — and no user-specified rules govern it — `review_rules` returns "No user rules provided."*
