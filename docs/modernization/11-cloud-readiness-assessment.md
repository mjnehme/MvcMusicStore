
# 11 — Cloud Readiness Assessment

## 1. Purpose, scope and authoring contract

### 1.1 What this document is

An assessment of whether the three ASP.NET MVC applications in this repository can run on a managed cloud platform as they stand, and of what must change before they can. It is one of the five supporting assessment records. It consumes deliverable [01 — Architecture Overview](01-architecture-overview.md) and deliverable [02 — Dependency Inventory](02-dependency-inventory.md), and it feeds deliverable [12 — Migration Blockers](12-migration-blockers.md) alongside deliverables 09 and 10.

Its organizing rule is that **every blocker states its target-state replacement, not only the problem**. A cloud readiness assessment that stops at the obstacle tells a reader what is wrong and leaves them no further forward; each of the eight blockers in section 3 therefore closes with a named replacement and a pointer to the deliverable that owns the design detail.

### 1.2 What this document is not

It is not a hosting recommendation, and it is not a design. It changes nothing in the applications it assesses, provisions nothing, and decides nothing that another deliverable owns. Section 1.4 lists the four decisions that are deliberately **not** made here.

It is also not a security assessment. Several findings below touch the same evidence deliverable 09 examines — the connection strings, the plaintext administrator credential, the absent transport protection — but the question asked here is different: not "is this safe?" but "does this work when the filesystem is ephemeral, the instance count is greater than one, and there is no domain controller?" Where the two overlap, the security consequence is cross-referenced to 09 and not restated.

### 1.3 The no-modification constraint

The user directed **"Do not make code changes initially"**, and the environment setup instructions attached to this project independently restate the same gate: *"Do not modify code until assessment and modernization plan are approved."* Two inputs agreeing on it is why no defect named below is repaired here.

No Azure resource is provisioned by this work — no resource group, App Service plan, Container Apps environment, Azure SQL instance, Key Vault or managed identity. **This is a document.** Every target-state replacement below is a *specification* for a separately approved implementation phase, not an action taken. Naming the replacement is required of this document; creating it is forbidden to it, and both halves matter.

**Restores and builds were performed, and they wrote into the checkout.** They were **unqualified historical restore and build operations, and their only relevance here is the gitignored residue they produced**: none of them recorded the fields a build outcome has to carry — tool versions, the restore source used, the configuration built, the per-edition outcome, the warning and error counts — so none of them is build evidence, and nothing in this document rests on one. **Every build outcome and build status is deliverable [10](10-build-and-deployment-requirements.md)'s**, on 10's own evidence and under 10's own qualifications (10 §1.4, §3.2, §5.4); in particular **MVC 5's build assessment remains blocked pending a Windows verification run**, and neither a restore that ran nor an output directory that existed may be read here as a verified application build. The Windows-and-Visual-Studio readiness result of section 3.8.4 does not depend on those runs either: it rests on inspection of the committed project files and on 10's toolchain diagnosis, as that section's citations show. What the runs left behind was **eight gitignored trees** — the restored `packages/` payload under `src/MVC4` and `src/MVC5`, and a `bin/` and `obj/` pair beside each of the three projects. **No extent for them is stated here**: the trees are gone, nothing immutable was retained of them, and a `bin`/`obj` extent is a property of the host that supplied the referenced assemblies and of the path the checkout sat at rather than of this repository — 10 §1.4 and its appendix A own that reasoning, along with the restored-payload extent that does reproduce. All eight were removed once the assessment had finished with them, and that is **a statement about a moment, not a durable property of the checkout**: an ignored tree returns the moment a build or a restore runs again, including one belonging to concurrent work rather than to this assessment, so keeping the checkout clear of them is a standing rule on whoever commits this assessment — 10 §1.4 states it and carries the per-tree record and the four-command check, which are not restated here. No tracked file was touched at any point.

**Why the obvious check cannot carry that on its own.** `bin/` and `obj/` [.gitignore:1-2] are ignored, and so are the two nested `packages` payloads — but by `Packages/` [.gitignore:33], **not** by the `packages/*` rule at [.gitignore:15], whose interior separator anchors it to the repository root and therefore cannot reach a directory under `src/`. `Packages/` matches a directory of that name at any depth, and it matches these lowercase ones **only because `core.ignorecase` is `true` on the verification host**: evaluated case-sensitively, `git -c core.ignorecase=false check-ignore --no-index -v src/MVC4/packages/x` reports nothing and exits 1, so **on a case-sensitive host nothing in `.gitignore` excludes the nested `packages` trees at all**. Deliverable [04](04-dotnet8-migration-strategy.md) §A.6 carries that analysis and it is not restated here. The readiness reading, which is this document's to make, is that it is the same class of hazard as the path-casing mismatch of section 3.7: an ignore rule written in one case against a directory spelled in another resolves differently on the two filesystems, so the repository's own hygiene tooling is not portable — section 3.7.2 reaches the identical finding from the `nuget.exe` rule. So `git status --porcelain` reports an empty working tree while a hundred megabytes of generated output sits in it — and a tracked-file diff cannot see ignored content either. An attestation resting on either alone attests to something it never looked at. **Four commands are therefore required, and on the committed checkout they have to hold together:** `git status --porcelain`, which returns no lines; `git status --porcelain --ignored`, which also returns no lines and is the only one of the four that looks at ignored content; `git clean -ndX`, which returns no output, so nothing ignored is left to remove; and the tracked diff against the pre-assessment baseline, `git diff --name-status ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD`, which returns exactly thirteen rows, every one an `A` for a file under `docs/modernization/`, with no `M` or `D` against any existing file. The first three describe the working tree and are therefore evidence only of the checkout they are run in — non-empty while uncommitted edits are in flight — while the fourth is a property of the committed history. **Both ends of that fourth command's range are named deliberately, and it is the only range this document uses.** The left side is the immutable pre-assessment revision `ea2552d6eda7c20e9477a512e5c615665618cf35` — the last commit before this assessment began, and the revision at which every source path cited below resolves, byte-identical to the delivered tree. The right side is `HEAD`, the delivery commit the reviewer has checked out; it is named as `HEAD` rather than as a hash because a document cannot cite the commit that creates it. All four are repeated in section 6 with the output each **must produce**, which is not the same claim as output this document observed: the two ignore-aware commands will legitimately report content in any checkout where a build or a restore has run since, so their empty result is the pre-commit condition owned by whoever commits this assessment ([10 §1.4](10-build-and-deployment-requirements.md)) rather than a property a reader can expect to reproduce on demand.

**The fourth command's two endpoints are not the same kind of thing, and the asymmetry is deliberate rather than an unfinished pin.** The **start** is pinned to a full hash — `ea2552d6eda7c20e9477a512e5c615665618cf35`, the last commit before this engagement — because a baseline that moves would let the diff shrink without anyone noticing. The **end** is `HEAD`, and it has to be: **no document can contain the hash of the commit that adds it.** That hash exists only once the commit does, which is after these thirteen files reach their final content, so any literal written here would necessarily name an *earlier* commit and would assert a range that excludes the very work being attested. A reader who wants both ends pinned resolves the tip once on their own checkout — `git rev-parse HEAD` — and substitutes the result for `HEAD`. **The substitution changes none of the four values the command must report**: exactly thirteen rows, all thirteen of them `A`, none of them anything else, and none outside `docs/modernization/`.

### 1.4 Authoring contract, and the absence of user rules

`review_rules` returns exactly **"No user rules provided."** There is therefore no project rule to name, summarize or comply with, and none is invented in its place. The absence is not licence to lower the bar; this document is held to enterprise-standard assessment practice and to four contracts that stand where rules would:

1. **Every as-is claim carries an inline `[<path>:<locator>]` citation** at the point the claim is made, repository-relative and resolving in the checkout. There is no trailing reference list, because a citation collected at the end cannot be checked against the sentence it supports.
2. **A repository-wide claim carries its reproducing command** adjacent to the claim. Most of the findings below are *absences* — no key configuration, no security header, no logging, no filesystem write — and an absence has no line to point at. The command is its evidence, and it is the stronger form because a reader can re-run it. Section 6 collects them all.
3. **Repository evidence is primary.** The Technical Specification is cited only as a secondary cross-reference, alongside a repository citation and never instead of one.
4. **Every claim names the edition or editions it holds in.** The repository ships three independent applications, not one application with three configurations (deliverable 01 §1.5). A claim about "the application" without an edition qualifier would be unverifiable here.

**Reproducing the commands on this host.** The commands quoted throughout are POSIX shell forms. The verification host is Windows; they were executed through the Git-for-Windows `bash` bundled on the host, from the repository root, and the outputs quoted are the outputs observed there. One practical note, because it changes how a byte total must be computed: **`bc` is not installed on this host**, so every sum below is taken with `awk '{s+=$1} END {print s}'`.

Markdown line length follows the corpus convention recorded in [`README.md` § 10 Markdown line-length convention](README.md#10-markdown-line-length-convention).

### 1.5 What this document does not own

Five decisions are owned elsewhere. This document states the *requirement* and its *consequence*, then points at the owner. This is not deference for its own sake — a requirement restated in different words downstream reads as a second decision, and a reader who finds two picks one.

| Decision | Owner | What this document does instead |
| --- | --- | --- |
| **Hosting target and deployment model** | [06 — Azure Hosting Recommendations](06-azure-hosting-recommendations.md) | Names the primary and secondary targets by cross-reference only. The choice is **not re-argued here**, and nothing below should be read as reopening it. |
| **Caching — whether the per-request layout aggregate is cached, where it is held, its lifetime and its invalidation** | [05](05-aspnet-core-migration-approach.md) §8.2 for the mechanism; [06](06-azure-hosting-recommendations.md) §6.4 for its hosting consequence | Records the readiness consequence of the uncached aggregate: which tier it loads first (section 3.8.2). It names **no cache and no placement**, and states no sizing or coherence requirement, because the mechanism is 05's and the sizing that follows from it is 06's. |
| **The observability and telemetry mechanism** | [06](06-azure-hosting-recommendations.md) | Records the *absence* of any instrumentation today (section 3.8.3) and states that a health surface and structured logging are required. It does **not** name the collection approach. |
| **Where the data-protection key ring lives**, what each candidate platform's default ring does, its protect-at-rest configuration, rotation policy and slot/revision isolation | [06](06-azure-hosting-recommendations.md) | States the requirement for persistent shared key storage, the consequence of not having it, whether that requirement is advisable or mandatory on each shape, and the three properties an external store adds (section 3.2) — citing 06 for the platform defaults themselves. The location and the policy are **not decided here**. |
| **Per-edition build outcomes** | [10 — Build and Deployment Requirements](10-build-and-deployment-requirements.md) | Records that a build requiring Windows and Visual Studio *is itself* a cloud-readiness result (section 3.8.4) and cites 10 for the outcomes. It does **not** restate the diagnosis. |

Three further facts are consumed rather than owned here: the security posture of the credentials and connection strings belongs to [09 — Security Assessment](09-security-assessment.md); the performance debt of the per-page aggregate and the tracked-yet-ignored binaries belong to [08 — Technical Debt Register](08-technical-debt-register.md); the EF Core and configuration transitions belong to [05 — ASP.NET Core Migration Approach](05-aspnet-core-migration-approach.md).

---

## 2. Readiness summary

Eight blockers and three favourable findings. Severity is assessed against a managed platform target specifically — an item is Critical where the application would be incorrect or unrunnable as hosted, High where it would be operationally unsound, Medium where it constrains a hosting option rather than the application.

| # | Blocker | Editions | Severity | Target-state replacement |
| --- | --- | --- | --- | --- |
| 3.1 | Cart identity held in in-process session; no shared key material | all three | Critical | Session over a SQL-backed distributed cache, shared data-protection keys, affinity as an interim measure only |
| 3.2 | No shared key material: auto-generated keys are machine-local, so nothing one instance protects is readable by another | all three | Critical | A shared key ring — already the platform default within one App Service slot; an external persistent store as a deliberate override for slot-swap continuity, protection at rest and container hosting, with the location owned by 06 |
| 3.3 | Local file-based database storage with no cloud analogue — file attachment in MVC 5 and MVC 4; in MVC 3 an in-process `.sdf` engine whose file is absent from the checkout, plus a host-inherited credential store | all three, three different engines | Critical | Managed SQL service with encryption in transit, reached without file attachment and without host-level provider configuration |
| 3.4 | Windows-authentication connection strings presenting a domain identity | MVC 5, MVC 4 | Critical | Managed identity for data-plane authentication — identity model owned by 06 |
| 3.5 | Configuration and secrets embedded in `Web.config` | all three | High | Platform configuration with secret references, read through a configuration abstraction |
| 3.6 | Plain HTTP, no HSTS, no security response header of any kind | all three | High | HTTPS enforced at the platform edge, HSTS, an explicit security-header set |
| 3.7 | Filesystem path casing mismatch | MVC 5 (demonstrated) | High — gates one hosting option | Repository-wide casing audit, completed **before** a case-sensitive hosting option is viable |
| 3.8 | Statefulness beyond session, and no observability whatsoever | all three | High | Health surface and structured logging — mechanism owned by 06 |
| 4.1 | `HttpContext.Current` appears nowhere | all three | *Favourable* | No static ambient-context coupling to unwind |
| 4.2 | No raw SQL execution anywhere | all three | *Favourable* | No SQL-dialect review to perform |
| 4.3 | No outbound service dependency, no inter-project reference | all three | *Favourable* | Nothing to re-point at a cloud endpoint |

**The shape of the result.** The blockers cluster in infrastructure coupling — session, keys, local database storage, host authentication, transport — and not in application logic. Sections 4.1 to 4.3 are the reason that distinction holds: the code does not reach for a static context, does not write to the filesystem, does not execute raw SQL and does not call anything outbound. What has to change is almost entirely *how the application is composed and hosted*, which is a bounded, largely mechanical body of work. An assessment listing only obstacles would misrepresent that.

---

## 3. Blockers, each with its target-state replacement

### 3.1 The cart identity lives in in-process session — and the precise consequence is narrower than it first appears

This is the finding most easily overstated, so it is stated carefully. Deliverable 01 §6.6 describes the mechanism and hands the statefulness consequence here.

#### 3.1.1 Evidence: five session access sites per edition

The mechanism is identical in all three editions — same constant, same resolution logic, same two access paths, five sites each — though the line numbers differ, so each edition is cited on its own lines.

The cart key is held under a compile-time constant, `CartSessionKey = "CartId"` [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:19], and resolved by `GetCartId(HttpContextBase context)` [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:161]. Four of the five sites are inside that method, reached through **`HttpContextBase.Session`**:

1. The null check that decides whether a key already exists [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:163].
2. For an authenticated request, the session slot is set to the **login name** [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:167].
3. Otherwise a fresh **`Guid.NewGuid()`** [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:172] is written as a string [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:175].
4. The stored value is returned as the cart key [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:179].

The fifth site is in the sign-in path and reaches session the other way, through the MVC **`Controller.Session`** property, writing the user name into the same slot after the cart rows have been reassigned [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:39].

The same five sites, per edition, each cited with its own full path and line so no cell depends on a shorthand locator inherited from a neighbouring column:

| Edition | Constant `CartSessionKey` | `GetCartId(HttpContextBase)` | The four `HttpContextBase.Session` sites | The `Controller.Session` site |
| --- | --- | --- | --- | --- |
| MVC 5 | [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:19] | [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:161] | [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:163], [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:167], [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:175], [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:179] | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:39] |
| MVC 4 | [src/MVC4/MvcMusicStore/Models/ShoppingCart.cs:19] | [src/MVC4/MvcMusicStore/Models/ShoppingCart.cs:161] | [src/MVC4/MvcMusicStore/Models/ShoppingCart.cs:163], [src/MVC4/MvcMusicStore/Models/ShoppingCart.cs:167], [src/MVC4/MvcMusicStore/Models/ShoppingCart.cs:175], [src/MVC4/MvcMusicStore/Models/ShoppingCart.cs:179] | [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:60] |
| MVC 3 | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:15] | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:166] | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:168], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:172], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:180], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:184] | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:22] |

Five sites per edition, fifteen in total:

```bash
# five per edition: four in the cart model, one in the account controller
for e in src/MVC5/MvcMusicStore src/MVC4/MvcMusicStore \
         src/MVC3/MvcMusicStore-Completed/MvcMusicStore; do
  grep -c 'Session\[' $e/Models/ShoppingCart.cs $e/Controllers/AccountController.cs
done
# -> 4 and 1 for each of the three editions
```

**The two key forms matter for what follows, and the difference between them is durability rather than recoverability.** A signed-in visitor's cart key is their login name [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:167] — a value recomputable from the authentication ticket on any instance, so it survives everything short of the account being renamed. An anonymous visitor's cart key is a `Guid.NewGuid()` [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:172] that is written into the session slot [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:175] and stored **nowhere else**: it is never written to a cookie of its own, never rendered into a page and never persisted against anything the client holds.

That is not the same as being unrecoverable on the next request, and the difference decides how the loss should be described: a cart that disappears on every request would be a functional defect, while a cart that disappears on four specific events is a statefulness constraint. The **session-ID cookie** the framework issues identifies the session, so a returning request carrying it finds the same session slot and the same GUID — for as long as that session exists, on the instance that holds it. In-process session state with the 20-minute idle timeout of section 3.1.2 therefore recovers an anonymous cart across ordinary browsing, and only these four events destroy the link:

1. **Session expiry** — 20 minutes idle by default, sliding, so a visitor who leaves the tab and returns after lunch comes back to an empty cart.
2. **Process or application-pool recycle** — the slot lives in the worker process, so a recycle, an overlapped restart or a redeploy discards it even though the same machine and the same client cookie remain.
3. **Instance loss** — scale-in, a platform update or a failure takes the whole store with the instance, per section 3.1.3.
4. **Cutover with no session continuity** — the new runtime cannot read the old process's in-memory slots at all, per section 3.1.4.

In every one of the four, the `Cart` rows keyed by the GUID remain in the database while the browser-to-GUID link does not, so the rows become unreachable rather than deleted — which is why they are orphaned data to be reported and cleaned up rather than a recoverable cart.

#### 3.1.2 Evidence: session storage and key material are both framework defaults

No edition declares a `<sessionState>` element or a `<machineKey>` element in any of its application configuration files. Both are therefore whatever the framework defaults to — in-process session storage, and **key material the repository neither declares nor controls**:

```bash
# 15 application config files; how many declare either element -> 0
git ls-files -- '*.config' '*.Config' | grep -v '/packages/' | grep -v '\.nuget/' \
  | xargs grep -lE '<sessionState|<machineKey' | wc -l
# -> 0     (denominator: the same pipeline without the xargs stage -> 15)
```

**What the absent `<machineKey>` actually means, stated precisely rather than as "keys are regenerated every time".** With no element declared, `System.Web`'s default is auto-generation with per-application isolation, and the generated material is **persisted** rather than freshly minted on every process start — a claim that it is regenerated per process would not survive a reviewer. The mechanism, and the three properties that follow from it, are owned and stated once by section 3.2.1; this section records only the readiness fact they produce, and adds no second description of where the material is held.

**And the point that matters is not the regeneration cadence — it is that nothing in the repository pins the value.** Key identity is therefore a property of the host rather than of the deployment, which has four consequences that hold whatever a given host happens to do: the material cannot be assumed shared across instances, cannot be inventoried, cannot be rotated deliberately, and cannot be isolated per environment. Any host behaviour that *does* share it is a platform property the application neither declares nor can rely on. Deliverable [09](09-security-assessment.md) owns the security consequence of unprotected and undeclared key material; recorded here it is a **portability** fact: the deployment does not carry its own key identity, so it cannot state what happens to a cookie when the instance behind it changes.

#### 3.1.3 The precise consequence

Multiple instances **can** run today, behind session affinity — which Azure App Service enables by default. Affinity pins each client to one instance, so the in-process session slot for that client is found on every request and the cart resolves correctly. The naive reading of this finding, that in-process session prevents running more than one instance, is wrong and a reviewer would catch it.

What in-process session forecloses is **resilient** scale-out. Three specific properties are lost:

- **Instance loss is user-visible data loss.** When an instance is recycled, scaled in, moved during a platform update or simply fails, its session store goes with it. Every anonymous visitor pinned to that instance loses the only copy of their cart key, and their cart rows are orphaned per section 3.1.1.
- **Every routine platform event becomes a customer-facing event.** A deployment, a plan scale-down, a configuration change that restarts the worker — all ordinary operations — each discard live anonymous carts.
- **Load distribution is by client, not by request.** Affinity is a constraint on the load balancer, so traffic cannot be spread evenly and a heavy client cannot be rebalanced.

Affinity also depends on the client returning the platform's affinity cookie, so a client that discards it, or an API-style caller that never stores it, is not pinned at all.

#### 3.1.4 Target-state replacement — three controls, all three required

Each addresses a different failure, and any one alone leaves a hole:

1. **Session backed by a distributed cache.** Session state moves out of the worker process into a SQL-backed distributed cache, so any instance can serve any request and an instance loss costs no cart. Deliverable [05](05-aspnet-core-migration-approach.md) owns the registration and the session-handling transition; deliverable [06](06-azure-hosting-recommendations.md) owns the provisioning of the cache table, its creation ordering and the principal that creates it.
2. **A key ring every instance resolves.** The session cookie that carries the session id is itself protected by the data-protection key ring, so distributed session is bounded by the key ring's reach. Section 3.2 states what each hosting shape supplies by default and what still has to be arranged, and section 3.2.3 scopes the dependency to the shapes where it bites rather than asserting it universally.
3. **Affinity documented as an interim measure, to be switched off once the first two are in place.** Affinity is the reason multiple instances work today — for the session slot *and* for the machine-local key material of section 3.2.1 — so it must remain until session and keys are shared, and it must then be explicitly disabled — otherwise the platform keeps pinning clients and the resilience the first two controls bought is never actually realized. Leaving it on indefinitely is the failure mode where the work is done and the benefit is not obtained.

A fourth item is a consequence rather than a control: **cutover does not carry anonymous carts across**, because the browser-to-GUID link only ever existed in the old process's memory. That is an approved delta owned by deliverable [05](05-aspnet-core-migration-approach.md), not a defect introduced by the replacement above.

### 3.2 Data-protection key material — "framework-provided" is not "configured"

Two frameworks are in play here and their defaults differ, so each is stated on its own primary source: what `System.Web` does today (3.2.1), and what the target framework does on the hosting shapes deliverable [06](06-azure-hosting-recommendations.md) is choosing between (3.2.2). Getting either wrong changes the target-state requirement — a false "ephemeral by default" claim would justify the replacement for a reason that does not exist — so neither is asserted from reputation.

#### 3.2.1 Today: auto-generated keys are persisted per machine, and the defect is cross-machine consistency

**Scope, stated first, because the two frameworks' defaults are routinely blurred and the blur changes the finding.** This subsection describes **only the three editions as they stand, running on `System.Web`**: auto-generated `<machineKey>` material, persisted per machine and isolated per application. What the **target** framework's key ring does by default — on a host it does not recognise, on Azure App Service and on a container-native platform — is section 3.2.2's subject, recorded there against its primary sources, and the two candidate platforms' behaviours are hosting facts deliverable [06](06-azure-hosting-recommendations.md) owns per section 1.5. Neither is restated here, and nothing in this subsection is evidence about either.

The separation is load-bearing rather than housekeeping: collapsing today's behaviour and the target platforms' defaults into one "per-instance and ephemeral" statement is wrong in three ways at once — it overstates the risk on one target platform, understates it on the other, and attributes to the application as it stands a default belonging to a framework it does not run on.

**Today's fact, which is the one this document is asserting, stated precisely because the imprecise version points at the wrong failure.** There is no key configuration of any kind in any edition — the same command as section 3.1.2 returns `0` for `<machineKey>`, and no edition declares any alternative key store. What applies instead is the framework's documented `<machineKey>` default, `AutoGenerate,IsolateApps` on both the validation and the decryption key, and its three properties are the ones that matter here:

1. **The key is generated once, not per process.** The framework generates random key material and **the operating system holds it outside the worker process**, in a machine-local store the platform protects at rest — documented for the `machineKey` auto-generate default as the Local Security Authority secret store. *Which* store a given host uses is a property of that host rather than of this repository, and nothing below depends on it; what the three properties rest on is that the material outlives the process and never leaves the machine.
2. **It is therefore stable across an application-pool recycle.** A recycle, a worker crash and an ordinary restart all leave the key in place, so a cookie issued before one is still decryptable after it. **The failure mode is not process lifetime**, and recording it as though it were would point the remediation at the wrong control — at restart behaviour rather than at the machine boundary of property 3.
3. **It is isolated per application and scoped to one machine.** `IsolateApps` derives a distinct key per application from the application's identity, and nothing in the mechanism transmits that key anywhere. Two instances of the same application on two machines therefore hold two *different* keys, and the framework's documented remedy for a web farm is to set the keys **manually and identically** on every server — which no edition does.

**So the readiness conclusion is unchanged, and property 3 rather than property 2 is what carries it.** Key material is machine-scoped, so **nothing one instance issues is readable by another**, and **the readiness conclusion does not depend on which target is chosen**: the target needs an explicitly persisted, shared key ring on either platform, for the two *different* reasons section 3.2.2 records against their primary sources — reasons that belong to the target framework's defaults and not to the behaviour described above. A managed platform scales by adding *instances*, which is exactly the boundary a machine-local key does not cross; that it would survive a recycle on any one of them buys nothing.

#### 3.2.2 On the target platform the default is better than that — and still not sufficient

Section 3.2.1's framing applies here: the stack is provided, and what still has to be decided is where the ring lives. The hosting defaults are genuinely capable; what they are not is uniform across hosting shapes, and the requirement below depends on which shape is selected. Away from a platform the framework recognises, its own default is a **local, filesystem-based ring under the host's profile directory** — unshared between hosts and lost with that directory — so it is the hosting shape, and not the framework, that decides what still has to be arranged. That baseline and both rows below rest on the citation given after the table.

| Hosting shape | Default key-ring behaviour | Consequence for this application |
| --- | --- | --- |
| **Azure App Service** | Keys are persisted to `%HOME%\ASP.NET\DataProtection-Keys`. That folder is backed by network storage and is synchronized across all machines hosting the app; it supplies the key ring to **all instances of an app within a single deployment slot**, and it survives restarts. Keys are **not protected at rest** there. | Cross-instance authentication-cookie, anti-forgery and session-cookie decryption **works by default within one slot** — this is not a failure mode to design around. Two things are still open: the ring is unprotected at rest, and **separate deployment slots do not share a key ring**, so a slot swap leaves the app unable to decrypt data protected under the previous slot's ring and signs users out of cookie authentication. |
| **A container with no persistent volume** — what a container-native platform's replicas are unless a volume or an external provider is configured | If none of the platform-detection conditions matches, keys "aren't persisted outside of the current process", and when the process shuts down all generated keys are lost. The documented guidance for containers is a persistent volume or an external provider. | This is where an ephemeral, per-replica key ring genuinely lives: each replica protects with material only it holds, and a revision replaces every replica. Persistent shared key storage is **mandatory** here, not advisable. |

Both rows: [Microsoft Learn, *Data Protection key management and lifetime in ASP.NET Core*, <https://learn.microsoft.com/aspnet/core/security/data-protection/configuration/default-settings> — verified 2026-08-28]. The App Service row is stated identically, under the heading *Data Protection key ring and deployment slots*, in [Microsoft Learn, *Deploy ASP.NET Core apps to Azure App Service*, <https://learn.microsoft.com/aspnet/core/host-and-deploy/azure-apps/> — verified 2026-08-28].

Two further properties of the default are requirements rather than failures, and both belong in the record because they are what the override in section 3.2.4 is actually bought with. Keys have a 90-day lifetime and are rolled automatically, so there is key *rotation* but no **governed** rotation — no operator-stated policy, no store an operator can inspect, back up or restore, and no record of when material changed. And pointing data protection at a specific repository *disables* automatic encryption of keys at rest unless at-rest protection is re-enabled by configuration, so the store and its at-rest configuration have to be chosen together. Same citation, *Data Protection key management and lifetime in ASP.NET Core*, verified 2026-08-28.

#### 3.2.3 The ordering constraint this document owns, scoped to where it bites

Distributed session (section 3.1.4, control 1) is reached through a session cookie that the key ring protects, so control 1 can deliver nothing the key ring does not already support. That dependency is real, but it is **conditional on the hosting shape** and is stated that way rather than absolutely:

- **App Service, within one deployment slot.** The ring is shared by default, so distributed session delivers its benefit as soon as it is registered. No key-store override is needed for this case to work, and claiming otherwise would misstate the platform.
- **Across an App Service slot swap.** The new slot cannot read cookies protected by the old one, so live sessions and sign-ins do not survive the swap even though the session rows in the shared cache do. Distributed session does not address this; a slot-independent ring does.
- **A container platform with no persistent key storage.** The ring does not survive a replica, so distributed session buys nothing until key storage is arranged. Here, and only here, control 1 without control 2 does not work at all.

Section 5 records this as a property of the controls rather than a schedule; deliverable [03](03-modernization-roadmap.md) owns the sequencing that acts on it.

#### 3.2.4 Target-state replacement — a deliberate override, not a repair of a broken default

**Persistent shared key storage in an external provider, adopted as a deliberate override of a working platform default.** The justification is the three properties the default does not supply, and stating them accurately is the point — the requirement does not rest on instances being unable to share keys, because within one App Service slot they can:

1. **Slot-swap and revision continuity.** An external ring is slot-independent, so the swap a zero-downtime release depends on does not sign every user out. The documented slot-independent options are Azure Blob Storage, Azure Key Vault, a SQL store and Redis.
2. **Protection at rest.** The App Service default ring is explicitly not protected at rest; an external store can be encrypted at rest and access-controlled to the application's own identity — with the at-rest configuration set explicitly, per section 3.2.2.
3. **One governed store.** A ring an operator can locate, back up, restore and audit, under a rotation policy that is stated rather than inherited from a framework default.

Deliverable [06](06-azure-hosting-recommendations.md) owns **where the ring lives**, its protect-at-rest configuration, its rotation policy, and the slot-or-revision isolation that stops a staging environment decrypting production cookies. Those are hosting decisions and this document does not take them. What it does fix is the requirement's *status*: advisable on App Service, where it buys the three properties above, and **mandatory on a container-native option**, where the platform default persists nothing.

For the **un-ported** application the equivalent control is different in kind: an explicit `<machineKey>` with fixed `validationKey` and `decryptionKey` values, shared across instances, per section 3.2.1. That is an edit to a tracked configuration file and therefore falls under the approval gate of section 1.3 — it is named here, not made — and the hosting analysis for the un-ported application is owned by [06](06-azure-hosting-recommendations.md).

### 3.3 A file-attached database has no cloud analogue

#### 3.3.1 Evidence: three editions, three different engines

Every edition reaches its data through a local database *file*, and no managed database service offers an equivalent for any of the three arrangements. **They are three cases and not one rule, though**, and stating them as one — "every edition attaches a database file from its own directory" — is wrong for MVC 3 in two separate ways, so each case is stated on its own evidence:

1. **MVC 5 and MVC 4 attach a *configured and committed* database file.** Both name `AttachDbFilename` in the connection string — MVC 5 for both of its stores [src/MVC5/MvcMusicStore/Web.config:12-13], MVC 4 for its credential store [src/MVC4/MvcMusicStore/Web.config:16] and its catalog store [src/MVC4/MvcMusicStore/Web.config:20] — and both ship the files those strings point at, which are among the ten committed binaries under `App_Data` counted in section 3.3.2. So the database really is inside the tracked tree and inside anything published from it.
2. **MVC 3's catalog is SQL Server Compact and its database file is absent from the checkout.** Its single connection string names `MvcMusicStore.sdf` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:57] under `providerName="System.Data.SqlServerCe.4.0"` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:58], and **no `.sdf` is tracked at `HEAD` and none exists in the working tree** — an absence, so it carries the commands below rather than a line. The file is therefore created on first run by an in-process engine rather than attached from the payload.
3. **MVC 3's identity storage is host-inherited and unknown from the repository.** Its `web.config` switches role management on with `<roleManager enabled="true" />` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:15] and selects Forms authentication [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:26] while declaring no membership provider, no role provider and no `LocalSqlServer` connection string — again an absence, evidenced by the command below. So `Membership` and `Roles` resolve through the machine-level ASP.NET SQL providers against the *host's* configuration, and which store they reach is a property of the machine rather than of this repository.

| Edition | Catalog store | Credential store | Storage mechanism |
| --- | --- | --- | --- |
| MVC 5 | LocalDB `MSSQLLocalDB`, `AttachDbFilename=\|DataDirectory\|\MvcMusicStore.mdf` [src/MVC5/MvcMusicStore/Web.config:13] | a separate file-attached Identity database via `DefaultConnection` [src/MVC5/MvcMusicStore/Web.config:12] | file attachment, both stores, files committed |
| MVC 4 | `(LocalDB)\v11.0` with `AttachDbFilename` [src/MVC4/MvcMusicStore/Web.config:19-21] | a separate file-attached SimpleMembership database, `Data Source=(LocalDb)\v11.0` [src/MVC4/MvcMusicStore/Web.config:13] with `AttachDBFilename` [src/MVC4/MvcMusicStore/Web.config:16] | file attachment, both stores, files committed, on a **retired instance name** |
| MVC 3 | **SQL Server Compact 4.0** — `Data Source=\|DataDirectory\|MvcMusicStore.sdf` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:57] with `providerName="System.Data.SqlServerCe.4.0"` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:58] | **not declared** — `<roleManager enabled="true" />` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:15] and Forms authentication [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:26] with no provider and no `LocalSqlServer` string, so it resolves from machine configuration (deliverable 01 §8.3) | **no attachment**: an in-process engine creating its `.sdf` on first run, with **no `.sdf` in the checkout**, plus a host-inherited credential store |

The two MVC 3 absence claims range over the repository and the working tree rather than over a line, so each carries its command:

```bash
# MVC 3's catalog file exists in neither the index nor the working tree
git ls-files | grep -ic '\.sdf$'                        # -> 0
find . -iname '*.sdf' -not -path './.git/*' | wc -l     # -> 0

# ...and MVC 3 declares no provider and no membership connection string
grep -nE '<membership|<roleManager|<profile|<providers|LocalSqlServer' \
  src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config
# -> 15:    <roleManager enabled="true" />
#    and nothing else: the feature is switched on with no <providers> of its own,
#    no <membership> element anywhere, and no LocalSqlServer connection string
```

Three distinct problems, not one repeated three times. MVC 5 attaches files to a LocalDB instance. MVC 4 does the same but names `v11.0`, a SQL Server 2012-era instance its own README contradicts. MVC 3 does not use SQL Server for its catalog at all: SQL Server Compact is an in-process engine reading a local `.sdf`, with no managed-service counterpart of any kind and no supported provider on the target framework — deliverable [12](12-migration-blockers.md) owns that as a no-successor construct.

**The deployment-payload consequence is MVC 5's and MVC 4's, and it holds exactly as stated for them.** For those two, `|DataDirectory|` resolves to the application's own `App_Data` folder, which makes the database part of the deployment payload. On a managed platform the application directory is a deployment artifact, not durable storage: it is replaced on deploy and not shared between instances. Two instances would attach two different copies of the same database and diverge, and shipping `App_Data` into a managed platform sends data into a directory that will be replaced (section 3.3.2).

**MVC 3's consequence is different in kind, and it is a cloud-readiness blocker in its own right.** Nothing is shipped, because there is no `.sdf` to ship. What the application depends on instead is host-level configuration that no repository artifact declares: a machine-wide SQL Server Compact 4.0 installation to create and read the catalog file, and a machine-level membership and role provider with its own connection string for the credential store. Both are properties of the machine, and a managed platform offers nowhere to put either — there is no host to install a provider onto and no machine-level configuration to inherit from — while any file the in-process engine did create would land in the same replaceable application directory. So MVC 3's data layer cannot be assessed as configured-and-portable at all; it would have to be re-declared before hosting is a question. Verification of the inherited machine-level provider on a supported Windows runtime is owned by deliverable [10](10-build-and-deployment-requirements.md).

#### 3.3.2 Evidence: the operational weight this leaves in the repository

Fourteen database binaries are committed, totalling **43,376,640 bytes**:

```bash
git ls-files | grep -icE '\.(mdf|ldf)$'
# -> 14
git ls-files | grep -iE '\.(mdf|ldf)$' | xargs -d '\n' stat -c '%s' \
  | awk '{s+=$1} END {print s}'          # bc is not installed on this host
# -> 43376640
```

| Directory | Files | Bytes | Matched by an ignore rule? |
| --- | --- | --- | --- |
| `src/MVC3/MvcMusicStore-Assets/Data/` | 4 | 18,079,744 | **No rule matches** — these four were never excluded |
| `src/MVC4/MvcMusicStore/App_Data/` | 6 | 12,779,520 | `App_Data/` [.gitignore:32] |
| `src/MVC5/MvcMusicStore/App_Data/` | 4 | 12,517,376 | `App_Data/` [.gitignore:32] |
| **Total** | **14** | **43,376,640** | 10 ignored-yet-tracked, 4 never excluded |

**The set splits 10 and 4, and the two halves are different facts.** `.gitignore` lists `App_Data/` [.gitignore:32], and that rule matches exactly the **10** binaries under an `App_Data` directory — six in MVC 4, four in MVC 5. An ignore rule cannot untrack a file already added, so those ten are tracked *despite* being excluded, which is what makes them debt rather than a decision. The remaining **4**, all under `src/MVC3/MvcMusicStore-Assets/Data/`, are matched by **no rule in `.gitignore` at all**: `ASPNETDB.MDF`, `MvcMusicStore.mdf`, `MvcMusicStore_log.ldf` and `aspnetdb_log.ldf` sit in a `Data/` directory the ignore file never mentions, so they were never excluded in the first place — arguably the worse of the two, because nothing ever expressed an intention to keep them out.

The distinction is checkable per file, and it needs `--no-index` to be checkable at all — `git check-ignore` skips tracked paths by default, as section 3.7.2 explains:

```bash
git ls-files | grep -iE '\.(mdf|ldf)$' | while read -r f; do
  r=$(git check-ignore --no-index -v "$f" || true)
  [ -z "$r" ] && echo "NO-MATCH  $f" || echo "$r"
done
# git separates the rule from the path with a tab; shown here as spaces
# -> NO-MATCH  src/MVC3/MvcMusicStore-Assets/Data/ASPNETDB.MDF
#    NO-MATCH  src/MVC3/MvcMusicStore-Assets/Data/MvcMusicStore.mdf
#    NO-MATCH  src/MVC3/MvcMusicStore-Assets/Data/MvcMusicStore_log.ldf
#    NO-MATCH  src/MVC3/MvcMusicStore-Assets/Data/aspnetdb_log.ldf
#    .gitignore:32:App_Data/  src/MVC4/MvcMusicStore/App_Data/MvcMusicStore-work.mdf
#    .gitignore:32:App_Data/  src/MVC4/MvcMusicStore/App_Data/MvcMusicStore.mdf
#    .gitignore:32:App_Data/  src/MVC4/MvcMusicStore/App_Data/MvcMusicStore_log-work.ldf
#    .gitignore:32:App_Data/  src/MVC4/MvcMusicStore/App_Data/MvcMusicStore_log.ldf
#    .gitignore:32:App_Data/  src/MVC4/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20120831200627.mdf
#    .gitignore:32:App_Data/  src/MVC4/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20120831200627_log.ldf
#    .gitignore:32:App_Data/  src/MVC5/MvcMusicStore/App_Data/MvcMusicStore.mdf
#    .gitignore:32:App_Data/  src/MVC5/MvcMusicStore/App_Data/MvcMusicStore_log.ldf
#    .gitignore:32:App_Data/  src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205.mdf
#    .gitignore:32:App_Data/  src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205_log.ldf
# -> 4 unmatched, 10 matched by .gitignore:32
```

Deliverable [08](08-technical-debt-register.md) owns both entries and deliverable [09](09-security-assessment.md) owns the exposure, since three of these files are credential stores. The cloud-readiness consequence is narrower and belongs here: every clone, build agent and deployment package carries 41 MiB of database files that the target hosting model has no use for, and a deployment that copies `App_Data` to a managed platform ships data into a directory that will be replaced. The ten-versus-four split matters to that consequence too — an `App_Data`-shaped exclusion in a publish profile or a container ignore file would omit the ten and still carry MVC 3's four.

#### 3.3.3 Target-state replacement

**A managed SQL service, reached over the network with encryption in transit, with no file attachment anywhere in the connection string.** The database stops being part of the deployment payload and becomes an external dependency addressed by hostname, so all instances share one authoritative copy and a redeployment cannot replace it. That replacement also closes MVC 3's separate case (section 3.3.1): both of its stores become explicitly declared external dependencies, so neither an in-process engine's first-run file nor a machine-level membership provider remains a runtime dependency the repository never states.

Two consequences follow and are owned elsewhere: schema and data have to get there, which is the EF Core transition and the data-migration workstream owned by deliverable [05](05-aspnet-core-migration-approach.md); and the instance has to be provisioned with a separation between the identity that applies schema changes and the identity the application runs as, owned by deliverable [06](06-azure-hosting-recommendations.md). MVC 3's provider retirement is owned by deliverable [12](12-migration-blockers.md).

### 3.4 Windows-authentication connection strings present an identity that will not exist

#### 3.4.1 Evidence

Every SQL Server connection string in the two SQL Server editions authenticates with the Windows identity of the worker process:

- MVC 5, catalog store: `Integrated Security=True` [src/MVC5/MvcMusicStore/Web.config:13], and the Identity store likewise [src/MVC5/MvcMusicStore/Web.config:12].
- MVC 4, credential store: `Integrated Security=SSPI` [src/MVC4/MvcMusicStore/Web.config:15]; catalog store: `Integrated Security=True` [src/MVC4/MvcMusicStore/Web.config:21].

MVC 3 declares no credentials at all for its catalog store [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:55-59] — SQL Server Compact is in-process and has no authentication step — and its credential store is a host-configured SQL Server store resolved from machine configuration rather than the repository (section 3.3.1, deliverable 01 §8.3). Whether *that* connection presents a Windows identity is therefore a property of the host and is **unverified**: [10 §13.2](10-build-and-deployment-requirements.md) item 2 carries it, so this document leaves it out of the blocker above rather than asserting an authentication mode nobody has observed.

#### 3.4.2 Why this is not a configuration tweak

`Integrated Security=True` and `Integrated Security=SSPI` are the same instruction: authenticate to SQL Server as the Windows principal the process is running as. That works because a developer machine and a domain-joined server both have such a principal and the database trusts it.

A platform-hosted worker has no domain identity to present. There is no Windows account meaningful to a managed SQL service, and no trust relationship for one to exercise. So the **authentication mechanism changes**, not merely its parameters — a different credential type, obtained a different way, granted through a different authorization model. Editing the string in place has no correct value: there is no domain identity to name.

#### 3.4.3 Target-state replacement

**Managed identity for data-plane authentication**, which removes the stored-credential problem rather than relocating it: the platform issues the application an identity, the database grants that identity least-privileged data access, and no secret exists to store, rotate or leak.

Deliverable [06](06-azure-hosting-recommendations.md) owns the identity model, the grants, and the caveat that applies if the application is hosted before it is ported — an un-ported application cannot authenticate this way as written, and 06 owns that analysis. It is not reproduced here.

### 3.5 Configuration and secrets are compiled into the deployment

#### 3.5.1 Evidence: configuration lives in the file that ships

`Web.config` carries the connection strings [src/MVC5/MvcMusicStore/Web.config:11-14] and the application settings, including an administrator username and password held as **plaintext values in tracked source** [src/MVC5/MvcMusicStore/Web.config:16-17]. MVC 4 ships the same two settings [src/MVC4/MvcMusicStore/Web.config:25-26]. The values are not reproduced in this document; they are cited by location, and deliverable [09](09-security-assessment.md) owns the security finding and both editions' startup-provisioning paths.

The cloud-readiness problem is structural and separate from the security one. Configuration that lives in a file inside the deployment payload can only be changed by redeploying, which means an environment cannot be configured without rebuilding for it, and the same artifact cannot be promoted from a test environment to production. A managed platform's model is the inverse: one artifact, configuration supplied by the environment.

#### 3.5.2 Evidence: the transform files carry exactly one active behaviour

The six XDT transform files look like an environment mechanism and very nearly are not one. Across all six there are 15 `xdt:Transform` occurrences, of which exactly **3 are active** — one per edition, the same transform, at the same line:

```bash
# for each XDT file: total xdt:Transform occurrences vs. occurrences
# surviving with XML comment blocks stripped
for f in $(git ls-files -- '*.Debug.config' '*.Release.config'); do
  printf '%s total=%s active=%s\n' "$f" "$(grep -c 'xdt:Transform' "$f")" \
    "$(python -c "import re,sys;s=open(sys.argv[1],encoding='utf-8-sig').read();print(re.sub(r'<!--.*?-->','',s,flags=re.S).count('xdt:Transform'))" "$f")"
done
# -> each Web.Release.config: total=3 active=1
# -> each Web.Debug.config:   total=2 active=0
```

The one active transform is `<compilation xdt:Transform="RemoveAttributes(debug)" />` [src/MVC5/MvcMusicStore/Web.Release.config:18], identical at line 18 of all three editions' `Web.Release.config` [src/MVC4/MvcMusicStore/Web.Release.config:18], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Web.Release.config:18]. Every other transform in all six files is commented-out template text.

**This one transform is a build-and-publish concern, not a runtime environment setting**, and conflating the two is the specific error to avoid here. It removes the `debug` attribute from the `<compilation>` element at publish time — a property of how the artifact is *built*. It does not select an environment, does not choose a configuration source and does not control production error behaviour. Deliverable [05](05-aspnet-core-migration-approach.md) owns that distinction and where each half lands.

Relatedly, and often misread as the transform's purpose: **`customErrors` never appears as a live element anywhere in the repository.** The figure has to name its unit, because four different numbers are all true of the same evidence — **24 word occurrences, 12 matches of the literal `<customErrors`, 6 example opening elements, and zero live elements** — and quoting one as though it were another is the specific error to avoid. Per file the same four figures are **4, 2, 1 and 0**, evenly across all six.

**Why the middle two differ, stated because the 12 is the figure most easily misread as a count of elements.** Of the four word occurrences in each file, exactly two match the literal `<customErrors`: the prose reference to the `<customErrors>` section inside the explanatory sentence [src/MVC5/MvcMusicStore/Web.Release.config:21], and the example element that carries the transform attributes [src/MVC5/MvcMusicStore/Web.Release.config:25]. The other two do not — the bare words `customErrors section` later in the same sentence [src/MVC5/MvcMusicStore/Web.Release.config:22], and the closing tag `</customErrors>` [src/MVC5/MvcMusicStore/Web.Release.config:28], which fails the literal because that `<` is followed by `/`. So only one of each file's two literal matches is an actual opening element, giving **6** across the six files, and none of the four units is a count of *configured* behaviour. Deliverables [08](08-technical-debt-register.md) and [09](09-security-assessment.md) carry the same taxonomy. Three commands separate the first three units, and a fourth shows the per-file distribution so that no total conceals an uneven one:

```bash
# unit 1 — every occurrence of the word: opening tags, closing tags and prose mentions alike
git ls-files -- '*Web.Debug.config' '*Web.Release.config' | grep -v '/packages/' \
  | xargs grep -o 'customErrors' | wc -l
# -> 24     word occurrences, four per file

# unit 2 — matches of the literal '<customErrors': two per file, the prose mention and the
# example element. '</customErrors>' does not match it, because that '<' is followed by '/'
git ls-files -- '*Web.Debug.config' '*Web.Release.config' | grep -v '/packages/' \
  | xargs grep -o '<customErrors' | wc -l
# -> 12     literal '<customErrors' matches, two per file

# unit 3 — actual example opening elements: the literal followed by whitespace, which the
# prose mention '<customErrors>' does not satisfy
git ls-files -- '*Web.Debug.config' '*Web.Release.config' | grep -v '/packages/' \
  | xargs grep -oE '<customErrors[[:space:]]' | wc -l
# -> 6      example opening elements, one per file

# the same three units, per file
for f in $(git ls-files -- '*Web.Debug.config' '*Web.Release.config'); do
  printf '%s words=%s literal=%s opening=%s\n' "$f" \
    "$(grep -o 'customErrors' "$f" | wc -l)" \
    "$(grep -o '<customErrors' "$f" | wc -l)" \
    "$(grep -oE '<customErrors[[:space:]]' "$f" | wc -l)"
done
# -> words=4 literal=2 opening=1 for every one of the six files
```

**Zero is live — zero of the 24, zero of the 12 and zero of the 6.** Every occurrence sits inside an XML comment block, which is verifiable by reading a block rather than by counting: in `Web.Release.config` the comment opens at line 19, describes a `<customErrors>` replacement transform, carries the example opening element at line 25 and the closing element at line 28, and the comment closes at line 29 [src/MVC5/MvcMusicStore/Web.Release.config:19-29]. The same shape appears in the other five files, and the liveness claim ranges over all fifteen application configuration files rather than only the six XDT files, so it carries its own command:

```bash
# live <customErrors elements, comments stripped, across all 15 application configs
for f in $(git ls-files -- '*.config' '*.Config' | grep -v '/packages/' | grep -v '\.nuget/'); do
  n=$(python -c "import re,sys;s=open(sys.argv[1],encoding='utf-8-sig').read();\
print(len(re.findall(r'<customErrors', re.sub(r'<!--.*?-->','',s,flags=re.S))))" "$f")
  [ "$n" != "0" ] && echo "LIVE $n $f"
done
# -> (no output): not one of the 15 declares a live <customErrors> element.
#    The only files mentioning it at all are the six XDT files: 4 word occurrences each,
#    of which 2 match the literal '<customErrors' and 1 is an example opening element.
```

So no edition configures production error behaviour at all. That absence is a runtime decision the port must make explicitly rather than inherit, and deliverable [05](05-aspnet-core-migration-approach.md) owns it together with the error-view replacement.

#### 3.5.3 Target-state replacement

**Platform-supplied configuration with secret references, read through a configuration abstraction rather than from a file in the payload.** Non-secret settings come from the platform's own configuration, secrets are referenced from a secret store rather than held as values, and one built artifact is promotable across environments because nothing environment-specific is inside it. **The administrator credential is removed from source entirely** — not moved to another file, and not carried into the new settings.

Deliverable [05](05-aspnet-core-migration-approach.md) owns the configuration transition and the shape of the settings; deliverable [06](06-azure-hosting-recommendations.md) owns the secret-delivery mechanism.

### 3.6 Transport: plain HTTP, and not one security response header

#### 3.6.1 Evidence

The projects' own web settings serve over plain HTTP with no TLS port configured. MVC 5 enables IIS Express [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:18] with an **empty** `IISExpressSSLPort` element [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:19], and its browse URL is `http://localhost:43524/` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:285]. MVC 4 likewise leaves `IISExpressSSLPort` empty [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:19] with `http://localhost:4321/` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:350]. No edition configures TLS.

**MVC 3 does not lack the IIS Express properties — it explicitly switches IIS Express off**, and the distinction matters because the two readings imply different host targets. Its project sets `<UseIISExpress>false</UseIISExpress>` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:17] and `<UseIIS>False</UseIIS>` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:223], leaves `<IISUrl>` empty [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:227-228], and instead pins `<DevelopmentServerPort>26641</DevelopmentServerPort>` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:225]. So the committed local host for MVC 3 is the Visual Studio Development Server, by deliberate configuration rather than by omission — over plain HTTP on a fixed port, with neither IIS nor IIS Express in the path.

The `DevelopmentServerPort` element is not what distinguishes it, and saying so keeps the comparison honest: MVC 5 carries one too, at 43524 [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:283], as does MVC 4 at 5928 [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:348]. What differs is which host the project resolves to — MVC 5 sets `<UseIISExpress>true</UseIISExpress>` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:18] with `<UseIIS>True</UseIIS>` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:281], and MVC 4 the same [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:18], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:346], so both run under IIS Express and their `DevelopmentServerPort` value is inert; MVC 3 sets both flags to `false`, which is what makes its development-server port the operative one.

The cloud-readiness consequence is a portability finding, not a gap: the host the project is configured against is a development server that exists on **no** supported platform — not on IIS, not on IIS Express, and not on any managed platform's runtime — so MVC 3's committed launch configuration cannot be carried anywhere and would have to be replaced outright rather than re-pointed. It also means MVC 3 has no TLS setting to be empty, which is a different kind of absence from MVC 5's and MVC 4's empty `IISExpressSSLPort`: those two declare a TLS port and leave it unset, while MVC 3 targets a host for which the property does not apply.

More consequentially, **no edition contains any transport-security or security-header construct whatsoever** — not a redirection, not HSTS, not one response header. This is a provable absence, so it is command-backed:

```bash
# every tracked config, C# and Razor file, excluding restored packages
git ls-files -- '*.config' '*.Config' '*.cs' '*.cshtml' | grep -v '/packages/' \
  | xargs grep -liE 'requireSSL|httpsOnly|RequireHttps|Strict-Transport|customHeaders|X-Frame-Options|X-Content-Type-Options|Content-Security-Policy|Referrer-Policy|Permissions-Policy' \
  | wc -l
# -> 0    (zero files, in any of the three editions)
```

The probe deliberately includes `requireSSL` and `httpsOnly`, so it also establishes that no authentication or session cookie is marked secure anywhere. The *cookie-attribute* consequence of that is deliverable [09](09-security-assessment.md)'s finding; recorded here it corroborates that transport protection is absent as a category rather than merely unconfigured in one place.

#### 3.6.2 Why this is a readiness item and not only a security one

A managed platform terminates TLS at its edge and forwards to the application over the internal network. An application that has never been told it is behind such an edge gets two **scheme-dependent** things wrong: with HTTPS redirection and HSTS in force it redirects requests that already arrived over HTTPS at the front end, because the hop it observes is plain HTTP — the loop that closes around that is traced in section 3.6.3 — and it generates absolute URLs, redirect targets and links in rendered pages alike, from the scheme it *sees* rather than the scheme the client used. Neither produces a warning. So the port must be explicitly configured for edge-terminated TLS; it does not become correct by being hosted somewhere that offers HTTPS.

**Cookie `Secure` marking is deliberately not on that list.** It is decided by policy, not by the observed scheme: deliverable [05](05-aspnet-core-migration-approach.md) §6.1 sets the authentication cookie's `Cookie.SecurePolicy` to `Always`, and under that value the attribute is *always* marked, whatever scheme the request arrived on — unlike `SameAsRequest`, which is the value that tracks the observed scheme [Microsoft Learn, *CookieSecurePolicy Enum*, <https://learn.microsoft.com/dotnet/api/microsoft.aspnetcore.http.cookiesecurepolicy> — verified 2026-08-28]. That is precisely why `Always` is the value to hold behind a TLS-terminating edge. Section 3.6.3 records the requirement that follows, and it is a requirement about policy rather than about scheme recovery.

#### 3.6.3 Target-state replacement

**HTTPS enforced at the platform edge, HSTS enabled, and an explicit security-header set applied by the application.** Insecure requests are redirected rather than served; HSTS instructs browsers not to attempt plaintext; and the header set is chosen and declared explicitly instead of remaining absent.

**Honouring the forwarded scheme is *required*, and this readiness item does not close on that requirement being recommended.** The target must reconstruct the client's scheme so that **redirection and absolute-URL generation** follow the client's connection rather than the internal hop; deliverable [05](05-aspnet-core-migration-approach.md) §2.4 wires that into the ordered `Program.cs` composition, and deliverable [06](06-azure-hosting-recommendations.md) §10.1.4 owns the ingress trust model — the single mode, and the topology that makes believing a forwarded header sound — within which such a header may be believed at all. Neither is restated here. That is what makes the requirement matter rather than tidy: with HTTPS redirection and HSTS in force and the client's scheme unrecovered, the application redirects a request that already arrived over HTTPS at the front end and the browser loops — a failure mode that exists only once the application sits behind that front end, so nothing in local development surfaces it.

**The cookie requirement is a separate one, met by policy rather than by scheme recovery.** Every cookie the target issues must be marked `Secure` by an explicit policy instead of being left to a framework default, and deliverable [05](05-aspnet-core-migration-approach.md) sets all three: the authentication cookie (§6.1), the session cookie (§6.3) and the anti-forgery cookie (§7.1). All three are stated explicitly there rather than any being inherited, because the target framework's defaults are not the same for the three. This document is the consumer of that decision, not its author — 05 owns the options and the values.

**The item closes on three deployment-observed conditions, and not before — each with its own cause, and none of them satisfied by another.** Behind the platform front end the application observes HTTPS and serves **without a redirect loop**, *because* forwarded-header processing is configured and ordered ahead of the redirection middleware ([05](05-aspnet-core-migration-approach.md) §2.4); the cookies in that response carry **`Secure`**, *because* 05's three cookie policies (§6.1, §6.3, §7.1) are `Always` and mark the attribute unconditionally; and the trust the first condition rests on is **established rather than assumed**, by deliverable [06](06-azure-hosting-recommendations.md) §10.1.4's `G-INGRESS` **assertion 5** — nothing but the platform front end can reach the application at all — together with its **assertion 6**, that a forwarded chain a *client* supplies through the real front end is not honoured beyond the single hop the configured limit admits. Those two are the adversarial half, they are deployment-only because each needs the real front end and the real network boundary, and they are 06's to run.

**Assertion 2 is not among them, and its absence is the owner's trust model rather than a relaxation here.** `G-INGRESS` **assertion 2** — a spoofed forwarded pair from an *untrusted peer* being ignored — is **withdrawn rather than skipped**, for the reason 06 §10.1.4 records: the selected trust model leaves no untrusted-peer branch to exercise, so the assertion would fail a correct implementation of it. That reason is 06's and is not re-argued here; the property it was reaching for is what assertions 5 and 6 carry, which is why those two are the ones named above. Three conditions rather than four is therefore a consequence of the owner's model and not a reduction in what this item demands: forwarding could be configured while a cookie policy was left implicit, the policies could be set while the scheme was never recovered, and both could hold on a deployment the front end is **not** the only path to — the case assertion 5 exists to exclude, and the one this readiness item would otherwise have no way to see.

Deliverable [06](06-azure-hosting-recommendations.md) owns the hosting-level enforcement and the platform settings that back it. **The target cookie contract is deliverable [05](05-aspnet-core-migration-approach.md)'s** — the cookie inventory and the target attribute values, and the labelling of each as preserved or as deliberate hardening, for the authentication cookie (§6.1), the session cookie (§6.3) and the anti-forgery cookie (§7.1); those values are cited here and not restated. Deliverable [09](09-security-assessment.md) owns the **current-state** posture the target departs from — what each edition's cookies and session actually inherit today, per edition at §3.3 (MVC 5), §4.3 (MVC 4) and §5.3 (MVC 3). This document is the consumer of both and the author of neither.

### 3.7 Filesystem path casing — a two-character mismatch that gates a hosting option

#### 3.7.1 Evidence

The style bundle registers `~/Content/site.css` with a **lowercase** `s` [src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs:28]. The tracked file is `Site.css` with a **capital** `S`:

```bash
git ls-files 'src/MVC5/MvcMusicStore/Content/*'
# -> src/MVC5/MvcMusicStore/Content/Site.css
#    src/MVC5/MvcMusicStore/Content/bootstrap.css
#    src/MVC5/MvcMusicStore/Content/bootstrap.min.css
```

Exactly three tracked files, and the one the bundle names is not spelled the way the bundle names it. The adjacent include in the same bundle, `~/Content/bootstrap.css` [src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs:27], matches its file exactly — so this is a single-file inconsistency rather than a convention, which is precisely what makes it easy to miss.

**IIS resolves this; a case-sensitive filesystem does not.** The mismatch **does not manifest under the case-insensitive Windows filesystem assumptions the application is documented to run under** — Visual Studio 2022 with SQL Server LocalDB [src/MVC5/README.md:7-8], served by IIS Express [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:18] — because the lookup folds case and finds the file. It **would fail on a case-sensitive filesystem**, which is what a Linux App Service plan or a Linux container image provides: the file is simply not found.

**What is claimed here is the mismatch and the filesystem rule, not an incident history.** Whether this has or has not manifested in any past run or deployment is not statically observable, and this document has no deployment record to draw on, so no such claim is made in either direction. The predicted failure mode on a case-sensitive filesystem is the quiet kind: the application starts, pages return 200, and the site stylesheet is missing — no exception, no failed health check, nothing in a log that would exist anyway (section 3.8.3). That is why the audit of section 3.7.3 is a precondition rather than a post-deployment check.

#### 3.7.2 The risk is demonstrated, not theoretical — the repository's own ignore rules resolve differently on the two filesystems

`.gitignore` excludes `nuget.exe` in lowercase [.gitignore:28] while the tracked path is `src/MVC4/MvcMusicStore/.nuget/NuGet.exe`, with a capital N and G. Whether that rule *matches* is not a fixed fact about the repository: it depends on the case sensitivity of the filesystem the rules are evaluated on, and this is directly observable.

**First, the check has to be asked correctly.** `git check-ignore` **skips tracked files by default** and exits 1 without reporting anything, so exit 1 on a tracked path says nothing whatsoever about the rules — it says only that the path is in the index. Reading that exit code as "no rule matches" is the specific mistake to avoid, and `--no-index` is what removes the ambiguity by evaluating the rules against the path itself:

```bash
# the default form on a TRACKED path proves nothing: tracked paths are skipped
git check-ignore -v src/MVC5/MvcMusicStore/App_Data/MvcMusicStore.mdf; echo "exit=$?"
# -> (no output)
#    exit=1                 <- says "tracked", not "unmatched"

# --no-index evaluates the rules and reports the one that matches
git check-ignore --no-index -v src/MVC5/MvcMusicStore/App_Data/MvcMusicStore.mdf
# -> .gitignore:32:App_Data/  src/MVC5/MvcMusicStore/App_Data/MvcMusicStore.mdf
#    (rule and path are tab-separated; shown here as spaces)
```

**Second, asked correctly, the casing rule gives two different answers on the two filesystems.** On the verification host `core.ignorecase` is `true`, because the filesystem is case-insensitive, and git folds case when matching ignore patterns — so the lowercase rule *does* match, and the same rule evaluated case-sensitively does not:

```bash
git config --get core.ignorecase
# -> true
git check-ignore --no-index -v src/MVC4/MvcMusicStore/.nuget/NuGet.exe; echo "exit=$?"
# -> .gitignore:28:nuget.exe  src/MVC4/MvcMusicStore/.nuget/NuGet.exe
#    exit=0                       <- matches, because case is folded here
git -c core.ignorecase=false check-ignore --no-index -v \
    src/MVC4/MvcMusicStore/.nuget/NuGet.exe; echo "exit=$?"
# -> (no output)
#    exit=1                       <- the same rule, the same path, no match
```

The same split appears in the packages rule, which is why that rule is cited **with its case qualification** wherever it appears — here and in section 1.3 — and the command's own output is quoted alongside it rather than the line number standing alone: `git check-ignore --no-index -v src/MVC4/MvcMusicStore/packages/EntityFramework.5.0.0/EntityFramework.5.0.0.nupkg` reports `.gitignore:33:Packages/` on this host and reports nothing at all under `core.ignorecase=false`. By contrast the `App_Data/` rule [.gitignore:32] matches under both settings, because its spelling and the directory's agree — which is exactly the point being made.

**That is the finding.** The repository's hygiene configuration is not portable: an identical rule set, an identical index, and two different results according to the filesystem underneath. A rule written in one case and a path spelled in another is precisely the class of defect section 3.7.1 identifies in the style bundle, demonstrated here in the repository's own tooling rather than argued from principle. The executable is tracked either way — an ignore rule cannot untrack a file already added, per section 3.3.2 — so the tracking is not the observation; the case-dependent resolution is.

Deliverable [02](02-dependency-inventory.md) §5.1 records the client itself and its exact size; deliverable [08](08-technical-debt-register.md) records the tracking as debt. Neither is restated here.

#### 3.7.3 Target-state replacement

**A repository-wide path-casing audit, completed as a precondition** — not a cleanup task to be done afterwards. Every path string the application resolves at runtime must be compared against the tracked filename, character for character. Three classes of site are in scope:

- **Bundle registrations** — all five bundles [src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs:11-28], including the `{version}` and glob token forms whose expansion is also case-sensitive.
- **`@Url.Content` calls in the views** — four occurrences in MVC 5: [src/MVC5/MvcMusicStore/Views/Home/Index.cshtml:15], [src/MVC5/MvcMusicStore/Views/Store/Browse.cshtml:17], [src/MVC5/MvcMusicStore/Views/Store/Details.cshtml:10] and [src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:75]. Reproduced by `grep -rn '@Url.Content' src/MVC5/MvcMusicStore/Views/ | wc -l` → `4`. Three of them resolve a database-supplied album art URL rather than a literal path, so the audit must cover the *data* as well as the source: a stored path whose casing does not match a tracked file fails identically.
- **View and partial paths**, wherever a view is named as a string rather than resolved by convention.

**The audit is a precondition for a case-sensitive hosting option, and the sequencing is the point.** Deliverable [06](06-azure-hosting-recommendations.md) owns the hosting recommendation this gates; the obligation recorded here is that the audit must pass before that option is viable, because the failure it prevents is silent and would otherwise be discovered by a user rather than by a deployment.

### 3.8 Statefulness beyond session, and the absence of any observability

#### 3.8.1 Local filesystem writes: a genuinely favourable result

No code in any edition writes to the local filesystem:

```bash
git ls-files -- '*.cs' '*.cshtml' | grep -v '/packages/' \
  | xargs grep -lE 'File\.(Write|Create|Append|Copy|Move|Delete)|StreamWriter|Directory\.(Create|Delete)|FileStream|Server\.MapPath|Path\.GetTempPath' \
  | wc -l
# -> 0
```

Zero files, across all three editions. This matters more than it may appear: local filesystem writes are one of the two classic sources of instance-affine state, and on a platform with an ephemeral or shared filesystem they produce data loss or cross-instance contention. That whole category is **absent** — there is no upload directory, no file-based cache, no log file, no temp-file dependency. The only instance-affine state in the application is session (section 3.1), and the attached database files (section 3.3) which are configuration rather than code. Recording this as a favourable finding is not padding: it removes an entire workstream that a typical application of this vintage would require.

#### 3.8.2 Per-request work that makes the database the first thing to scale

The genre menu runs an uncached nested aggregate — a `Sum` over each genre's albums' order-detail quantities, ordered and taken nine at a time [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:46-52] — and because it is a child action rendered from the shared layout [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:25] it executes on **every page request**, as does the cart summary rendered beside it [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:26]. Deliverable [01](01-architecture-overview.md) §5.3 carries both child actions and their call sites.

The readiness consequence is narrow and worth separating from the performance one. This is not instance-affine state, so it does not block scale-out; there is no in-process cache holding it, which is why nothing here breaks when instances multiply. What it does is multiply database round-trips by request volume, so the managed database becomes the scaling bottleneck and the cost driver **before** the compute tier does — which inverts the usual assumption that adding instances is what a traffic increase costs.

**This document does not choose a remedy, and deliberately names no cache.** Whether the aggregate is cached, **where it is held, under what key, with what lifetime, how it is invalidated and how it behaves when the store is unreachable** is owned by deliverable [05](05-aspnet-core-migration-approach.md) §8.2, which decides the mechanism explicitly. Deliverable [06](06-azure-hosting-recommendations.md) §6.4 owns only the **hosting and sizing consequence** of where that store lands — its register row S7 says so in terms — and the performance debt itself, with its invalidation write paths, is owned by deliverable [08](08-technical-debt-register.md) §5.2. The readiness fact recorded here — and the only one this document owns — is that the database, not the compute tier, is where this workload lands first, so capacity planning for the managed database must account for per-request layout traffic rather than for order and checkout volume alone.

#### 3.8.3 There is no observability of any kind

A managed platform's operational model assumes the application reports on itself. This one reports nothing at all. Five independent probes, all returning zero across all three editions:

```bash
SRC=$(git ls-files -- '*.cs' '*.cshtml' | grep -v '/packages/')
CFG=$(git ls-files -- '*.config' '*.Config' | grep -v '/packages/' | grep -v '\.nuget/')

echo "$SRC" | xargs grep -lE 'ILogger|log4net|NLog|Serilog|EnterpriseLibrary|Elmah' | wc -l   # -> 0
echo "$SRC" | xargs grep -lE 'TraceSource|System\.Diagnostics\.Trace|Trace\.Write|EventLog' | wc -l  # -> 0
echo "$CFG" | xargs grep -lE '<healthMonitoring|<system\.diagnostics|<trace ' | wc -l         # -> 0
echo "$SRC" | xargs grep -liE 'ActionResult (Health|Ping|Ready|Live|Status)\b' | wc -l         # -> 0
echo "$SRC" | xargs grep -lE 'PerformanceCounter|Metric|Telemetry|ApplicationInsights' | wc -l # -> 0
```

No logging abstraction, no logging framework, no tracing, no health monitoring configuration, no health or readiness endpoint, no metrics.

**A platform cannot assess the health of an application that exposes none.** This is the concrete operational consequence, and it compounds specifically with two findings above. Platform health probes fall back to whether the process accepts a TCP connection, so an application that is running but cannot reach its database is reported healthy and kept in rotation. And the casing failure of section 3.7 is silent *because* of this: a missing stylesheet produces no log entry, since there is nowhere for one to go. Combined with the blanket exception handler in the checkout path that deliverable [08](08-technical-debt-register.md) records, a failed order in production today would leave no trace anywhere.

**Target-state replacement.** Two requirements: **a health surface the platform can probe** — reporting readiness in terms of the dependencies that actually matter, the database above all, so that an instance which cannot serve is removed from rotation rather than left in it — and **structured logging emitted through a logging abstraction**, so that a failure produces a record with enough context to diagnose it. The **mechanism by which telemetry is collected and where it is sent is owned by deliverable [06](06-azure-hosting-recommendations.md)** and is deliberately not named here; what this document establishes is that the application must produce the signal in the first place, which today it does not.

#### 3.8.4 The build itself requires Windows and Visual Studio — which is a readiness result

The toolchain needed to build these projects is a cloud-readiness finding in its own right, not merely a developer-experience note. A build that requires Windows, Visual Studio Build Tools and a .NET Framework targeting pack constrains where the artifact can be produced: the build agent must be a Windows image, which narrows the hosted-agent choices, and it is the reason a container-native deployment of the un-ported application would need a Windows base image.

**This rests on the committed project files, not on any build run** (section 1.3). All three projects target .NET Framework — `v4.8` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:16], `v4.5` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:16] and `v4.0` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:15] — and each imports the Visual Studio web-application targets: MVC 5 and MVC 4 conditionally through `$(VSToolsPath)` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:272], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:337], and MVC 3 unconditionally from a Visual Studio 2010-era path [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:209]. A target framework that exists only on Windows plus an import that resolves only inside a Visual Studio installation is the whole of the claim; no build had to be run to read it.

Deliverable [10](10-build-and-deployment-requirements.md) owns the per-edition build outcomes and the toolchain diagnosis, including which checklist items could be satisfied on the verification host and which could not, and MVC 5's build status remains **blocked pending a Windows verification run** there. Those are cited, not restated. The readiness consequence recorded here is the one 10 does not carry: **the pipeline platform is constrained by the framework choice, and that constraint is lifted by the port rather than by the hosting decision** — a cross-platform target framework is what makes a Linux build agent and a Linux container base image available, and no amount of hosting configuration achieves it beforehand.

---

## 4. Favourable findings

An assessment that lists only obstacles overstates the work and is less useful for scoping it. Three verified facts materially reduce the difficulty of the port, and each is an absence — which is why each carries its command.

### 4.1 `HttpContext.Current` appears nowhere

```bash
git ls-files -- '*.cs' '*.cshtml' | grep -v '/packages/' \
  | xargs grep -l 'HttpContext\.Current' | wc -l
# -> 0
```

Zero files, across every `.cs` and `.cshtml` file in all three editions. This is the single most valuable finding in the document.

`HttpContext.Current` is the hardest `System.Web` coupling to unwind: a static ambient context reachable from arbitrary code at arbitrary depth, with no compile-time signal about which call paths depend on a request being in flight. Removing it usually means tracing every transitive caller of every method that touches it, and it is the reason many migrations of this kind stall. **It is simply absent here.**

Every context access is instead explicit and local: either an `HttpContextBase` parameter passed in — as `GetCartId(HttpContextBase context)` does [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:161], called with `this.HttpContext` from the controller [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:35] — or the MVC `Controller.HttpContext` / `Controller.Session` property inside a controller [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:39]. Both forms have direct equivalents in the target framework, and because the dependency is a visible parameter or an inherited member, the compiler identifies every site that needs attention.

### 4.2 No raw SQL execution anywhere

```bash
git ls-files -- '*.cs' '*.cshtml' | grep -v '/packages/' \
  | xargs grep -lE 'ExecuteSqlCommand|SqlQuery|SqlCommand|CommandText' | wc -l
# -> 0
```

Zero files. Data access is uniformly ORM-mediated LINQ in all three editions, so there is **no SQL-dialect review to perform** — no hand-written statement to re-verify against a managed database, no provider-specific syntax to translate, and no place where a query string could silently carry a construct the target platform rejects. This matters most for MVC 3, whose engine changes category entirely (section 3.3.1): even there, the queries are expressed against the ORM rather than against SQL Server Compact's dialect.

### 4.3 No outbound service dependency and no inter-project reference

```bash
git ls-files -- '*.csproj' | xargs grep -l '<ProjectReference' | wc -l
# -> 0
```

No project file in the repository declares a `<ProjectReference>`; each of the three applications is a leaf project with no internal dependency. Deliverable 01 §5.1 records the layering, and deliverable [02](02-dependency-inventory.md) records that no dependency in any edition is an outbound service client.

The readiness consequence: **there is no external endpoint to re-point, no service credential to migrate, and no network egress rule to plan.** The application's only outbound dependency is its database, which section 3.3 already addresses. There is no payment gateway, no mail service, no identity provider actually enabled — the social-login registrations are commented out, per deliverable 01 §9.3 — and therefore no integration surface whose cloud behaviour has to be assessed separately.

---

## 5. The consolidated target-state control set

Every replacement named above, in one place, with the deliverable that owns its design. This is a routing aid; no decision is taken here that is not already stated in the section cited.

| # | Control required | Addresses | Design owned by |
| --- | --- | --- | --- |
| 1 | Session over a SQL-backed distributed cache | 3.1 | 05 (registration), 06 (cache table provisioning and ordering) |
| 2 | A key ring every instance resolves: the platform default within one App Service slot, overridden to an external persistent store for slot-swap continuity, protection at rest and one governed store — mandatory on a container-native option | 3.1, 3.2 | 06 (location, protect-at-rest configuration, rotation, slot or revision isolation) |
| 3 | Session affinity retained as an interim measure, then explicitly disabled | 3.1 | 06 (platform setting), 03 (sequencing) |
| 4 | Managed SQL service, no file attachment, encryption in transit | 3.3 | 05 (EF Core and data migration), 06 (provisioning, DDL separation) |
| 5 | Managed identity for data-plane authentication | 3.4 | 06 (identity model and grants) |
| 6 | Platform configuration with secret references; credential removed from source | 3.5 | 05 (configuration transition), 06 (secret delivery), 09 (credential finding) |
| 7 | HTTPS enforced at the edge, HSTS, explicit security headers, forwarded-scheme handling for redirection and absolute-URL generation, and `Secure` cookies by explicit policy — **required, and not established by being recommended**: it closes on 05 §2.4's wiring plus three deployment-observed conditions, each with its own cause — no redirect loop because forwarding is configured and ordered ahead of redirection, `Secure` cookies because 05's three policies (§6.1, §6.3, §7.1) say `Always`, and the trust those rest on established by 06 §10.1.4's `G-INGRESS` assertions 5 and 6 rather than assumed, its withdrawn assertion 2 not required here (section 3.6.3) | 3.6 | 05 (composition, and the target cookie contract: inventory, attribute values and their preserved-or-hardened labels at §6.1, §6.3, §7.1), 06 (enforcement, ingress trust model and its assertions), 09 (the current-state posture the target departs from, §3.3, §4.3, §5.3) |
| 8 | Repository-wide path-casing audit, as a precondition | 3.7 | 06 (the hosting option it gates) |
| 9 | Health surface reporting dependency readiness | 3.8.3 | 06 (probe configuration) |
| 10 | Structured logging through a logging abstraction | 3.8.3 | 06 (telemetry mechanism) |

**Two ordering constraints are properties of the controls themselves** and are recorded here because they are readiness facts rather than schedule decisions; deliverable [03](03-modernization-roadmap.md) owns the sequencing that acts on them.

- **Control 1's benefit is bounded by the reach of the key ring, and the bound depends on the hosting shape** (section 3.2.3): distributed session is reached through a cookie the key ring protects. Within one App Service slot the platform already shares the ring, so control 1 delivers as soon as it is registered; across a slot swap, and on a container platform with no persistent key storage, it delivers nothing until control 2 is in place.
- **Control 3 must be reversed only after 1 and 2 are both live** (section 3.1.4): affinity is what allows a multi-instance deployment of the application as it stands to work at all — for the in-process session slot (section 3.1.2) and for the machine-local key material (section 3.2.1) alike — so disabling it earlier removes the property that is holding the deployment together.

---

## 6. Reproducibility appendix

Every command in this document, collected for re-execution. All are read-only: none writes to the working tree — the single `git clean` invocation is `-n`, a dry run that reports and removes nothing — and none contacts the network. Run from the repository root. POSIX forms, executed on this Windows host through the bundled Git-for-Windows `bash`. **`bc` is not installed here**, so byte totals use `awk`.

```bash
# --- Statefulness: session and key material (§3.1, §3.2) --------------------
# five session access sites per edition: four in the cart model, one in the controller
for e in src/MVC5/MvcMusicStore src/MVC4/MvcMusicStore \
         src/MVC3/MvcMusicStore-Completed/MvcMusicStore; do
  grep -c 'Session\[' $e/Models/ShoppingCart.cs $e/Controllers/AccountController.cs
done                                                            # -> 4 and 1, each edition

# the 15 application config files (the denominator for every config claim)
git ls-files -- '*.config' '*.Config' | grep -v '/packages/' | grep -v '\.nuget/' | wc -l   # -> 15

# ...of which how many declare <sessionState> or <machineKey>
git ls-files -- '*.config' '*.Config' | grep -v '/packages/' | grep -v '\.nuget/' \
  | xargs grep -lE '<sessionState|<machineKey' | wc -l                                     # -> 0

# the same absence stated the other way round (§3.2.1): no tracked .config mentions the
# element at all, so the packages filter changes nothing and the unfiltered form is empty too
git grep -n 'machineKey' -- '*.config' | grep -v '/packages/'                              # -> no output

# --- Committed database binaries (§3.3.2) -----------------------------------
git ls-files | grep -icE '\.(mdf|ldf)$'                                                    # -> 14
git ls-files | grep -iE '\.(mdf|ldf)$' | xargs -d '\n' stat -c '%s' \
  | awk '{s+=$1} END {print s}'                                                            # -> 43376640
# per-directory subtotals quoted in the §3.3.2 table
git ls-files | grep -iE '\.(mdf|ldf)$' \
  | while read f; do printf '%s\t%s\n' "$(dirname "$f")" "$(stat -c '%s' "$f")"; done \
  | awk -F'\t' '{c[$1]++; s[$1]+=$2} END {for (d in s) print c[d], s[d], d}' | sort -k3
# which of the 14 an ignore rule actually matches: 10 under App_Data/, 4 matched by nothing
git ls-files | grep -iE '\.(mdf|ldf)$' | while read -r f; do
  r=$(git check-ignore --no-index -v "$f" || true)
  [ -z "$r" ] && echo "NO-MATCH  $f" || echo "$r"
done            # -> 4x NO-MATCH under src/MVC3/MvcMusicStore-Assets/Data/; 10x .gitignore:32:App_Data/

# --- Configuration transforms (§3.5.2) --------------------------------------
for f in $(git ls-files -- '*.Debug.config' '*.Release.config'); do
  printf '%s total=%s active=%s\n' "$f" "$(grep -c 'xdt:Transform' "$f")" \
    "$(python -c "import re,sys;s=open(sys.argv[1],encoding='utf-8-sig').read();print(re.sub(r'<!--.*?-->','',s,flags=re.S).count('xdt:Transform'))" "$f")"
done                          # -> Release: total=3 active=1 (line 18); Debug: total=2 active=0

# customErrors, with every unit stated (§3.5.2): 24 word occurrences, 12 matches of the
# literal '<customErrors', 6 example opening elements, 0 live — per file 4, 2, 1 and 0
git ls-files -- '*Web.Debug.config' '*Web.Release.config' | grep -v '/packages/' \
  | xargs grep -o 'customErrors' | wc -l                    # -> 24  word occurrences (4/file)
git ls-files -- '*Web.Debug.config' '*Web.Release.config' | grep -v '/packages/' \
  | xargs grep -o '<customErrors' | wc -l                   # -> 12  literal matches (2/file)
git ls-files -- '*Web.Debug.config' '*Web.Release.config' | grep -v '/packages/' \
  | xargs grep -oE '<customErrors[[:space:]]' | wc -l       # -> 6   example openings (1/file)
# the per-file distribution behind those three totals
for f in $(git ls-files -- '*Web.Debug.config' '*Web.Release.config'); do
  printf '%s words=%s literal=%s opening=%s\n' "$f" \
    "$(grep -o 'customErrors' "$f" | wc -l)" \
    "$(grep -o '<customErrors' "$f" | wc -l)" \
    "$(grep -oE '<customErrors[[:space:]]' "$f" | wc -l)"
done                                    # -> words=4 literal=2 opening=1 for each of the six
# none live: every one is inside an XML comment block, e.g. Web.Release.config lines 19-29.
# The liveness claim ranges over all 15 application configs, so it is checked over all 15:
for f in $(git ls-files -- '*.config' '*.Config' | grep -v '/packages/' | grep -v '\.nuget/'); do
  n=$(python -c "import re,sys;s=open(sys.argv[1],encoding='utf-8-sig').read();\
print(len(re.findall(r'<customErrors', re.sub(r'<!--.*?-->','',s,flags=re.S))))" "$f")
  [ "$n" != "0" ] && echo "LIVE $n $f"
done                                                       # -> no output: zero live elements

# --- Transport and security headers (§3.6.1) --------------------------------
# the committed local host per edition: MVC 3 switches IIS Express off rather than lacking it
grep -n 'UseIISExpress\|UseIIS>\|IISExpressSSLPort\|DevelopmentServerPort\|IISUrl' \
  src/MVC5/MvcMusicStore/MvcMusicStore.csproj \
  src/MVC4/MvcMusicStore/MvcMusicStore.csproj \
  src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj
# -> MVC5 :18 UseIISExpress true, :19 empty IISExpressSSLPort, :281 UseIIS True,
#         :283 DevelopmentServerPort 43524 (inert), :285 http://localhost:43524/
#    MVC4 :18 UseIISExpress true, :19 empty IISExpressSSLPort, :346 UseIIS True,
#         :348 DevelopmentServerPort 5928 (inert), :350 http://localhost:4321/
#    MVC3 :17 UseIISExpress false, :223 UseIIS False, :225 DevelopmentServerPort 26641,
#         :227-228 empty IISUrl, and no IISExpressSSLPort element at all

git ls-files -- '*.config' '*.Config' '*.cs' '*.cshtml' | grep -v '/packages/' \
  | xargs grep -liE 'requireSSL|httpsOnly|RequireHttps|Strict-Transport|customHeaders|X-Frame-Options|X-Content-Type-Options|Content-Security-Policy|Referrer-Policy|Permissions-Policy' \
  | wc -l                                                                                  # -> 0

# --- Path casing (§3.7) -----------------------------------------------------
git ls-files 'src/MVC5/MvcMusicStore/Content/*'          # -> 3 files; Site.css has a capital S
grep -n 'site.css' src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs      # -> :28, lowercase
grep -rn '@Url.Content' src/MVC5/MvcMusicStore/Views/ | wc -l                              # -> 4
# the case-dependent ignore resolution of §3.7.2. --no-index is required: without it,
# check-ignore skips tracked paths and exits 1 without evaluating any rule
git config --get core.ignorecase                                       # -> true (on this host)
git check-ignore --no-index -v src/MVC4/MvcMusicStore/.nuget/NuGet.exe; echo "exit=$?"
                              # -> .gitignore:28:nuget.exe  <path>, exit=0: case folded, matches
git -c core.ignorecase=false check-ignore --no-index -v \
    src/MVC4/MvcMusicStore/.nuget/NuGet.exe; echo "exit=$?"
                              # -> no output, exit=1: same rule, same path, no case-sensitive match
git check-ignore -v src/MVC5/MvcMusicStore/App_Data/MvcMusicStore.mdf; echo "exit=$?"
                              # -> no output, exit=1 — because the path is tracked, not unmatched

# --- Statefulness beyond session, and observability (§3.8) ------------------
SRC=$(git ls-files -- '*.cs' '*.cshtml' | grep -v '/packages/')
CFG=$(git ls-files -- '*.config' '*.Config' | grep -v '/packages/' | grep -v '\.nuget/')
echo "$SRC" | xargs grep -lE 'File\.(Write|Create|Append|Copy|Move|Delete)|StreamWriter|Directory\.(Create|Delete)|FileStream|Server\.MapPath|Path\.GetTempPath' | wc -l  # -> 0
echo "$SRC" | xargs grep -lE 'ILogger|log4net|NLog|Serilog|EnterpriseLibrary|Elmah' | wc -l              # -> 0
echo "$SRC" | xargs grep -lE 'TraceSource|System\.Diagnostics\.Trace|Trace\.Write|EventLog' | wc -l      # -> 0
echo "$CFG" | xargs grep -lE '<healthMonitoring|<system\.diagnostics|<trace ' | wc -l                    # -> 0
echo "$SRC" | xargs grep -liE 'ActionResult (Health|Ping|Ready|Live|Status)\b' | wc -l                   # -> 0
echo "$SRC" | xargs grep -lE 'PerformanceCounter|Metric|Telemetry|ApplicationInsights' | wc -l           # -> 0

# --- Favourable findings (§4) ----------------------------------------------
echo "$SRC" | xargs grep -l 'HttpContext\.Current' | wc -l                                 # -> 0
echo "$SRC" | xargs grep -lE 'ExecuteSqlCommand|SqlQuery|SqlCommand|CommandText' | wc -l   # -> 0
git ls-files -- '*.csproj' | xargs grep -l '<ProjectReference' | wc -l                     # -> 0

# --- The constraint this work was held to (§1.3) ---------------------------
# Restores and builds ran while this assessment was written and left eight gitignored trees
# in the checkout: the packages/ payload under src/MVC4 and src/MVC5, and a bin/ and obj/
# pair beside each of the three projects. They were unqualified operations and none is build
# evidence; build outcomes and status are deliverable 10's (§1.3). All eight trees were
# removed, and no command below reads a build result. The attestation is these
# four commands together, because the first and the last are both blind to ignored content
# — bin/ and obj/ [.gitignore:1-2], and the nested packages trees by Packages/
# [.gitignore:33] rather than by packages/* [.gitignore:15], which is root-anchored and
# cannot reach them — and so cannot see a restored package payload or a
# build-output tree sitting in the working directory. Which rule covers which tree, and the
# case dependence of the Packages/ match (rule and path are tab-separated; spaces here):
git check-ignore --no-index -v src/MVC4/packages/x src/MVC5/packages/x \
    src/MVC5/MvcMusicStore/bin/x src/MVC5/MvcMusicStore/obj/x
# -> .gitignore:33:Packages/  src/MVC4/packages/x
#    .gitignore:33:Packages/  src/MVC5/packages/x
#    .gitignore:2:[Bb]in/     src/MVC5/MvcMusicStore/bin/x
#    .gitignore:1:[Oo]bj/     src/MVC5/MvcMusicStore/obj/x
git -c core.ignorecase=false check-ignore --no-index -v src/MVC4/packages/x; echo "exit=$?"
# -> (no output), exit=1: evaluated case-sensitively, the nested lowercase packages tree is
#    excluded by no rule at all — the portability point of §1.3 and §3.7.2
# The first three commands below read the working tree, so what follows each is the output it
# MUST produce at the committed checkout and not output recorded here; they are non-empty while
# uncommitted edits are in flight, and the two ignore-aware ones are non-empty wherever a build
# or a restore has run since -- which is the committer's condition to meet, not a reader's.
git status --porcelain
# -> (no output): no tracked file modified, added or deleted, and nothing untracked
git status --porcelain --ignored
# -> (no output): and nothing ignored either — the only one of the four that looks at it
git clean -ndX
# -> (no output): dry-run removal of ignored files finds nothing left to remove
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
# -> thirteen rows, every one an A (addition) under docs/modernization/, and nothing else:
#    no M and no D against any existing file. This command reports the index only; the
#    generated trees named above are covered by the three commands ahead of it.
git diff --name-status ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD | grep -c '^A'   # -> 13
git diff --name-status ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD | wc -l           # -> 13
git diff --name-status ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD \
  | grep -v 'docs/modernization/' | wc -l                                               # -> 0
```

---

## 7. Where each finding is consumed

| Finding | Section | Consumed by |
| --- | --- | --- |
| Session-held cart identity; resilient scale-out foreclosed | 3.1 | 12 (blocker), 05 (session transition), 06 (cache provisioning, affinity setting), 03 (sequencing) |
| No shared key material **today** — auto-generated `<machineKey>` material is machine-local, so nothing one instance protects is readable by another (3.2.1); on the **target** platform the default ring is shared only within a single App Service deployment slot and unprotected at rest there, and persists nothing at all on a container platform with no volume (3.2.2) | 3.2 | 12, 06 (key-ring location and policy, and what each candidate platform's default ring does) |
| Local file-based database storage; three engines; MVC 3's absent `.sdf` and host-inherited credential store; 14 binaries totalling 43,376,640 bytes, 10 ignored-yet-tracked and 4 never excluded | 3.3 | 12, 05 (EF Core, data migration), 06 (provisioning), 08 (tracked binaries), 09 (credential stores), 10 (per-edition topology and the open MVC 3 host-verification item) |
| Windows-authentication connection strings | 3.4 | 12, 06 (identity model), 09 (stored-credential posture) |
| Configuration and secrets in the payload; one active XDT transform | 3.5 | 05 (configuration transition, build-vs-runtime split), 06 (secret delivery), 09 (plaintext credential) |
| Plain HTTP; no HSTS; no security header of any kind; forwarded-scheme handling required and unverified; MVC 3 configured against the Visual Studio Development Server | 3.6 | 12, 05 (composition, and the target cookie contract at §6.1, §6.3, §7.1), 06 (edge enforcement, and the §10.1.4 ingress trust model whose assertions 5 and 6 close §3.6), 09 (the current-state cookie and session posture, §3.3, §4.3, §5.3), 10 (the MVC 3 local host) |
| Path-casing mismatch; the case-dependent `.gitignore` corroboration | 3.7 | 06 (the hosting option it gates), 08 §10.7 / F-08-28 §10.7 / F-08-28 (the ignore-rule mechanics and the ignore-rule mechanics and the `NuGet.exe` debt entry) |
| No local filesystem writes | 3.8.1, 4 | 07 (a workstream that is not needed) |
| Uncached per-request aggregate; the database scales before the compute tier | 3.8.2 | 08 (performance debt), 05 §8.2 (the caching mechanism — store, key, lifetime, invalidation and failure), 06 §6.4 (the sizing that follows) |
| No logging, tracing, health surface or metrics | 3.8.3 | 12, 06 (telemetry mechanism), 07 (net-new work) |
| Windows-and-Visual-Studio build constraint as a readiness result | 3.8.4 | 10 (build outcomes), 06 (build agent and base image) |
| `HttpContext.Current` absent; no raw SQL; no outbound dependency | 4.1–4.3 | 07 (scoping and effort), 05 (context and query transitions) |

### 7.1 The two finding-register rows this deliverable discharges — F-09-29 and F-09-30

The table above routes *outward*: a finding recorded here, and the deliverables that consume it. Two routes run *inward*, and they are recorded here so that each is checkable from the other end rather than only from 09's side. **[Deliverable 09](09-security-assessment.md)'s finding register assigns exactly two rows to this document — `F-09-29` and `F-09-30`** — and 09 §8.3 names where they belong: this document's transport and statefulness assessment. Both are security findings whose *readiness* consequence is this document's to state, which is why each appears above as a blocker paired with a target-state replacement rather than as a restatement of 09's posture.

| Register row | What it states | Where this document discharges it |
| --- | --- | --- |
| `F-09-29` | Nothing requires, redirects to or asserts HTTPS, and no edition emits a single security response header | **§3.6.** §3.6.1 establishes both halves on evidence — the empty `IISExpressSSLPort` and plain-HTTP browse URL [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:19], and the command-backed absence of every transport-security and header construct across all tracked config, C# and Razor files, which returns `0` files. §3.6.2 states the part that is this document's rather than 09's: an application that has never been told it sits behind a TLS-terminating edge gets scheme-dependent behaviour wrong, which is a readiness failure and not only a security one. §3.6.3 pairs it with the target-state replacement — enforced HTTPS, HSTS, an explicit header set and forwarded-scheme handling — and closes only on the three deployment-observed conditions named there, the third of which is 06 §10.1.4's `G-INGRESS` assertions 5 and 6 |
| `F-09-30` | No `<machineKey>` or key material anywhere, so the keys protecting authentication tickets and anti-forgery tokens belong to the host rather than to the deployment — unpinnable, unrotatable, and not valid on a second host | **§3.2**, and the scoping there is what carries it. §3.2.1 records that no edition declares key configuration of any kind, and states the three properties of what the framework then does: the key is generated **once** and held by the operating system, it is therefore **stable across an application-pool recycle**, and it is **isolated per application and scoped to one machine**. **Property 3, not property 2, is what carries the readiness conclusion**: nothing one instance issues is readable by another, so a cookie or an anti-forgery token does not survive a move between hosts. §3.2.2 shows the target platforms' defaults are better and still not uniform, and §5's control set carries the explicit shared key ring. The key-ring *store*, its protect-at-rest configuration, rotation and slot or revision isolation are deliverable [06](06-azure-hosting-recommendations.md)'s, per §1.5 |

**Neither row is acquired by being cited here.** Severity, security framing and remediation ownership are 09's; the readiness reading and the target-state replacement are this document's. The traversal terminates in both directions: from 09's register, each row's Consumers cell names `11`, and this sub-section names both rows by identifier; from here, `F-09-29` and `F-09-30` resolve to rows in 09 §8 whose Consumers cells name `11` alongside `06`. Both directions are checkable mechanically rather than by reading: deliverable [09](09-security-assessment.md) §9.5 compares the register's `(row, consumer)` pairs against the identifiers each consuming deliverable prints **in its designated closure section — which, for this deliverable, is this sub-section** — and the two pairs above are two of the 52 it reports. That check exits non-zero on any mismatch, so a rewording here that dropped either identifier out of this sub-section would fail it rather than pass quietly. Where the two ends disagree, 09 §8's cell governs, per 09 §8.3.

---

*Deliverable 11 of 13. Consumes deliverables [01](01-architecture-overview.md) and [02](02-dependency-inventory.md); feeds deliverable [12](12-migration-blockers.md) alongside 09 and 10. Index and requirement map: [README](README.md). Every count in this document is reproducible by the command stated beside it, and no tracked repository file outside `docs/modernization/` was created, modified or deleted in its production — the gitignored restore and build output that the assessment did produce was removed once the assessment had finished with it, which section 1.3 records as a statement about a moment rather than a durable property of the checkout, under the standing rule 10 §1.4 carries.*
