
# 09 — Security Assessment (as-is)

> **This document changes nothing.** It records the security posture of the three shipped
> MvcMusicStore editions as they exist in the checkout. Every defect below is **documented, not
> repaired** — including the ones it would take one line to fix. Section 1.3 states why.

---

## 1. Purpose, scope and authoring contract

### 1.1 What this document is

The security assessment of the three ASP.NET MVC applications in this repository, answering the
**"Security concerns"** analysis requirement. It records, **per edition**: authentication and
authorization posture; cookie, session, password and lockout policy; the Identity and account flows;
secret handling; anti-forgery emission versus validation; PII exposure; error disclosure; data
protection; and auditability. Section 6 then records what holds in all three.

Two of its findings are absences rather than defects, and for those it additionally states the contract
they are closed against — the security-event catalog of §6.8.1 and the personal-data governance control
set of §6.8.2. Section 1.2 bounds that exception and section 1.5 places it against the deliverables that
own the implementation.

It consumes deliverable 01 (architecture) and deliverable 02 (dependencies), and it feeds deliverable
12 (migration blockers) alongside 10 and 11; the two contracts above are consumed by 05 and 06 for
implementation and by 03 and 07 for sequencing and sizing.

### 1.2 What this document is not

It is not a remediation plan, a penetration test or a threat model. **It asserts no advisory
identifier at all, and neither does any other document in this assessment** — deliverable
[02 §8.2](02-dependency-inventory.md) states why (no scanner is part of this environment and the
repository holds no dependency-scanning configuration to cite) and gives the route by which a reader
obtains today's set from NuGet's own restore-time audit, with the bounds on what such a result
establishes; [02 §8.1](02-dependency-inventory.md) carries the aged-pin class with the exact pins, and
[02 §8.3](02-dependency-inventory.md) carries the scan the implementation phase owes. Where this
document draws a security conclusion from a dependency, it draws it from the pin and its position in
the code — repository facts — and never from an identifier or a severity nobody here measured. It does
not rank the editions against each other as products — MVC 4 and MVC 3 are retained read-only as
historical references and as the behavioural baseline, and neither is a migration target.

It is also **not the target-state security design**, with one bounded exception that section 1.5 draws
precisely. The design belongs to deliverables 05 and 06: the pipeline, the authentication and
anti-forgery policy, the Identity migration and the provisioning replacement are 05's, and the platform
mechanics — the log sink, the log-privacy policy, retention, key material, transport enforcement and
secret delivery — are 06's. What this document additionally owns is the **closure criteria of its own
findings**, and for the two findings that are *absences* rather than defects that means owning a
**contract**: §6.8.1 is the security-event catalog — sixteen event classes with their identifiers,
actors, targets, outcomes, severities and permitted fields — and §6.8.2 is the six-control
personal-data governance set with what counts as meeting each and who approves it. Both are contracts,
not implementations: they say what must be emitted and what must be governed, never how the pipeline
emits it, where the sink is, what the retention window is or which platform service holds a key. Those
remain 05's and 06's, and are cited here rather than restated.

### 1.3 The no-modification constraint — and why this deliverable feels it most

The user directed **"Do not make code changes initially."** The project's attached environment setup
instructions restate the same gate independently: **"Do not modify code until assessment and
modernization plan are approved."** The acceptance criterion is mechanical — after this work,
`git status --porcelain` must show exactly the new files under `docs/modernization/` and nothing else.

This is the deliverable under the greatest pressure to fix what it finds, because most of what it
finds is a one-line fix. It must not. No `Web.config` is edited to remove the committed credential.
No `[ValidateAntiForgeryToken]` attribute is added to the five POST actions that lack one. No verb
attribute is added to `AddToCart`. Remediation is prescribed by deliverable 05 and executed only
after approval.

One consequence is worth stating plainly, because it looks like a mistake and is not. **The committed
administrator credential's *value* is withheld from this document; its setting keys and its exact
source locators are not.** The finding is that a working credential sits in tracked configuration and
in git history, and that is established by the locators — [src/MVC5/MvcMusicStore/Web.config:16-17],
[src/MVC4/MvcMusicStore/Web.config:25-26] and [src/MVC5/README.md:75-85] — not by reprinting the
value here. A reader with the checkout confirms it in one command; a reader without it gains nothing
from a copy sitting in a document that is itself committed. Redaction therefore costs no
verifiability and lowers no severity: F-09-05 stays **Critical**, in both editions, with both
provisioning paths cited. Deliverable 08 §5.6 applies the same rule to the same credential, and this
document matches it deliberately rather than by coincidence. **Redaction is a property of this
document only** — rotating or obscuring the value in the repository would be a repository change,
which this section forbids.

The rule has one stated boundary, so its application is checkable rather than a matter of taste.
**A value is withheld when it can authenticate an account the running application creates**, and two
sets of literals meet that test. The first is the administrator credential of F-09-05, which both
editions create on first run and which the MVC 5 README documents as created automatically
[src/MVC5/README.md:77]. The second is MVC 3's hard-coded password-recovery question and answer of
F-09-17: **every** account the edition registers is created with those two literals
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:94], so they are
recovery credentials for every user the application has ever created rather than inert sample text.
Both sets are withheld the same way and for the same reason, and in both cases the **argument
positions, the setting keys and the exact locators are retained** — a reader with the checkout
confirms either in one command, and a reader without it gains nothing from a copy in a committed
document.

**The

**And withholding a value is not a remediation of it.** Committed authenticating material is treated
as **compromised** in both cases, because both are valid, both are in git history, and no edit to this
document changes either fact. So the remediation is **rotation at the deployment** — stated as a
closure criterion by §3.5 for the administrator credential and by §5.1 for the recovery literals —
while this document's own obligation stops at not adding a further copy. A reader who finds a
withheld value here should read it as "this document declines to be one more place the secret
exists", never as "the secret has been dealt with". rule binds every part of this document without exception, and the finding register of §8 is
named here because it is the part most likely to escape one.** A register row is a summary, and a
summary is where a withheld value is most tempting to inline "so that the row reads concretely" — so
the rule is stated for the register explicitly: **a row states what the value is, which argument
position or setting key carries it, and where to read it; never the value itself.** An earlier form of
F-09-17's row printed both recovery literals while §5.1 below redacted them, which is exactly the
failure this sentence exists to prevent — a document that republishes a credential it has already
committed to withholding has not withheld it, and a register is the most-read part of an assessment.
**One asymmetry remains, and it is deliberate rather than an exception:** the administrator *user
name* — the value of `DefaultAdminUsername`, `Administrator`
[src/MVC5/MvcMusicStore/Web.config:16], [src/MVC4/MvcMusicStore/Web.config:25] — and the *fact* that a
working plaintext password sits on the next line are both retained, because a user name authenticates
nothing on its own and the fact is the finding. The password's value is retained nowhere in this
document.

**Nothing else in the repository falls inside the rule**, so no other value in this
document is withheld: **no connection string in any edition carries a password** — MVC 5 and MVC 4
authenticate through Windows integrated security (§3.9, §4.8) and MVC 3's names a file rather than a
server (§5.8) — and the repository declares no `machineKey` or other key material to quote (§6.6, §9).

### 1.4 Authoring contract, and the absence of user rules

`review_rules` returns exactly **"No user rules provided."** No project rule constrains this
document, no rule forces any file into its scope, and none is invented in its place. Their absence is
not licence to lower the bar; the assessment's own contracts bind instead, and five govern this file:

1. **Every claim about the existing system carries an inline `[<path>:<locator>]` citation** at the
   point the claim is made, repository-relative and resolving in the checkout. There is no trailing
   reference list — a citation collected at the end cannot be checked against the sentence it
   supports. For a text file the locator is a line range, a heading or a configuration key path; for
   a **binary**, where no line range exists, it is the **evidence form and the artifact's size** —
   for example a string probe against a database file of a stated byte length — paired with the
   reproducing command in section 9.
2. **Every finding states which editions it holds in.** This document describes three different
   authentication stacks, so a claim about "the application" that silently holds in only one of them
   is not imprecise, it is wrong. Every finding in sections 3 to 6 carries an explicit
   **Editions** line, and the register in section 8 carries an editions column.
3. **Repository evidence is primary.** Technical Specification §6.4 describes the same three stacks,
   the same framework-default policy inheritance, the same per-edition anti-forgery coverage and the
   same absence of audit logging and transport protection; it is cited only as a secondary
   cross-reference, **alongside a repository citation and never instead of one**.
4. **Repository-wide claims — counts and absences — carry the command that reproduces them**,
   adjacent to the claim. A count has no single line to point at, so the command is its evidence, and
   it is the stronger form because a reader can re-run it. Section 9 collects them.
5. **Where a claim about framework behaviour can be tested, it is tested rather than asserted.**
   Section 4.9 is the case in point: the finding there is the one the repository establishes, and
   running MVC 4's own shipped assembly then **bounded** it — fixing what the framework's default
   rendering does with the disclosed object. A measurement of that kind qualifies a finding; it never
   stands in for one, and no finding in this document is downgraded or withdrawn on its strength.
   It can, however, **establish what a pinned assembly does on a path the repository's own code
   calls** — which is a repository-grounded claim, because both the call site and the pin are files in
   this checkout and only the callee's internals are unreadable from them. F-09-38 in §3.4 is the one
   finding of that shape, and it states its method, its counted result and the pin it exercised
   alongside the call site, so a reader can reproduce all three parts. No finding rests on a
   measurement without a call site and a pin to attach it to.

   **The two measurements in this document are retained to different depths, and each site states its
   own depth rather than a single claim covering both.** Section 1.3 forbids adding any file to this
   repository, so **neither harness is committed as a repository artifact** — both lived in a scratch
   directory outside the checkout. What each one leaves behind in *this document* is not the same:

   - **The rendering probe behind F-09-14 is published in full, in §9.3.** Its harness source is listed
     complete, together with the assembly under test and its file version, the compiler and runtime it
     was built and run on, the host, the date, the literal build and run commands, and the **verbatim**
     console output with its exit code. A reader who does not trust the §4.9 table can rebuild that
     measurement from this document alone. Publishing it is deliberate: it is the one measurement here
     that could not be made by reading a file, and a harness described rather than shown is a harness
     nobody can check.
   - **The counter probe behind F-09-38 is retained as a transcribed result**, and §3.4 says so at its
     own site: the package identity, the invocation shape, the four-step reproduction procedure and the
     observed counter values are retained; the harness source, the tool versions, the literal commands
     and the raw output are **not**.

   Two rules follow and both are applied at each site. First, **no finding is stated on the strength of
   a measurement alone**: each of F-09-14 and F-09-38 stands on its call site and its pin, with the
   measurement doing strictly the qualifying or corroborating job its own paragraph names, and each
   paragraph says which. Second, **the durable form of each measurement is its reproduction procedure,
   not its recorded numbers**: §9.3 carries that procedure for the rendering probe and §3.4 for the
   counter probe, a re-run is what a downstream deliverable relies on rather than these figures, and any
   implementation that needs the numbers as evidence re-runs the procedure and retains the complete
   output — source, tool versions, commands and raw output — as a build artifact outside this repository.

Markdown line length follows the corpus convention recorded in [`README.md` § 10 Markdown line-length convention](README.md#10-markdown-line-length-convention).

### 1.5 What this document does not own

One fact, one owner. This document owns **present posture, present risk, and the criteria its own
findings are closed against**. It does not decide the target state, and where a finding has a forward
decision attached, the decision belongs to the deliverable named below and is cross-referenced rather
than restated — a restatement in different words reads downstream as a second decision.

The one place that boundary needs stating rather than implying is F-09-32 and F-09-33, whose subject is
an *absence*. An absence has no line for another deliverable to fix, so recording it is not enough to
make it closable; §6.8.1 and §6.8.2 therefore state the **contracts** those two findings are closed
against — the event catalog and the personal-data governance control set. They are the closure criteria
of findings in this document, and they stop at the contract: emission mechanics and the log sink are 05's
and 06's respectively, the log-privacy policy and retention are 06 §9.2 and §9.5, the platform delivery
of any key material is 06 §8.4, and the workstream and sizing are 03's and 07's. Everything in the table
below is unaffected by that exception.

| Forward decision | Owned by |
| --- | --- |
| The anti-forgery target policy, and the decision to convert `AddToCart` to a token-protected POST | 05 |
| The Identity schema-and-data migration, and the explicit cookie/password/lockout policy that replaces today's inherited defaults | 05 |
| The administrator-provisioning replacement, and the cross-store sign-in ordering | 05 |
| The data-protection key store, and the observability and telemetry approach | 06 |
| The deployment-time migration principal and the separation of DDL from data-plane rights | 06 |
| Session statefulness and connection-string modernization as *cloud-readiness* concerns | 11 |
| Per-edition build outcomes and database prerequisites | 10 |
| The hygiene framing of the committed binaries, the debt severities and their owners | 08 |
| Dependency versions, pins and the restore audit output | 02 |
| Effort, the risk register and sequencing | 07, 03 |

### 1.6 How to read the findings

Sections 3, 4 and 5 are per-edition, so no finding can be edition-ambiguous by construction. Section
6 holds only what is true of all three. Section 7 records the controls that **are** present, which is
not politeness — an assessment that lists only failures cannot be calibrated, and several of the
gaps below are only legible against the places the same codebase gets it right. Section 8 is the
register, section 9 the reproducibility appendix.

Severity is assigned on exposure in a **hosted, internet-facing deployment**, which is the target
this assessment exists to serve. It is not assigned on exposure as a local tutorial, where several
Critical findings are harmless.

| Severity | Meaning |
| --- | --- |
| **Critical** | Direct loss of confidentiality, integrity or availability of credentials or customer data, exploitable as shipped |
| **High** | A missing control that a standard attack exercises directly, or privilege far in excess of need |
| **Medium** | A weakness requiring a precondition, or a control present but incomplete |
| **Low** | Hardening gap or defence-in-depth loss with no direct exploit path as shipped |

---

## 2. Posture at a glance

### 2.1 Three editions, three postures

Deliverable 01 §8 establishes the architecture of the three stacks; it closes by assigning their
security posture here. The one-line summary is that the three editions are **not** three points on
one hardening curve — each is missing a different set of controls, and the newest is not uniformly
the safest.

- **MVC 5** emits the most anti-forgery tokens, which is *not* the same as validating the most — its
  validation gap is the same five actions as MVC 4's, at the same line numbers (F-09-08, §3.7), and
  the wider emission is what makes a view-level audit misleading rather than what makes the edition
  safer (F-09-12, §4.7). It also ships a plaintext administrator credential and provisions it from an
  `async void` method whose failures nothing can observe.
- **MVC 4** ships the *same* plaintext credential and a connection string carrying enough privilege
  to create databases, and it is simultaneously the only edition whose administrator provisioning is
  correctly idempotent and the only edition that HTML-encodes a value it echoes back.
- **MVC 3** has no anti-forgery control in either direction, signs users out over `GET`, guards its
  administration surface with a role no code ever creates, permits over-posting where the other two
  forbid it, and cannot be assessed for password or lockout policy from the repository at all,
  because it declares no credential store.

### 2.2 Control coverage by edition

Every cell is evidenced in the section named in the row. A blank claim in this table is not
permitted — where a control's state cannot be determined from the repository, the cell says so.

| Control | MVC 3 | MVC 4 | MVC 5 | Detail |
| --- | --- | --- | --- | --- |
| Authentication mechanism | classic Membership, Forms | SimpleMembership, Forms | Identity 1.0, OWIN cookie | §5.1, §4.1, §3.1 |
| Credential store declared in the repository | **no — inherited from the host** | yes | yes | §5.1, §4.1, §3.1 |
| Password policy determinable from the repository | **no** | no — framework default | no — framework default | §5.3, §4.3, §3.3 |
| Lockout | **not determinable** | **none in the code path** | **none in the code path** — Identity 1.0 predates the feature; the shipped store's physical columns are **extraction-gated**, not asserted here (§3.3) | §5.3, §4.3, §3.3 |
| Sign-in failure *message* identical for unknown user and wrong password | yes | yes | yes | §5.4, §4.4, §3.4 |
| Sign-in *timing* equalised across the two failure branches | **not determinable — the provider is the host's (§5.1)** | **not established — the SimpleMembership path was not exercised (§4.4)** | **no — an unknown name is answered with no password verification at all (F-09-38)** | §5.1, §4.4, §3.4 |
| Authorization enforcement points (counting rule in §3.2) | **6** — but 2 of them protect one action and 1 can never be satisfied | **6** | **5** | §5.2, §4.2, §3.2 |
| Resource-level ownership check on an **account-management** action | **n/a — no external-login surface to guard (§5.10)** | **yes** — `Disassociate`'s owner comparison [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:126] | **n/a — the equivalent action binds the caller's own id into the framework call** [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:117] | §4.2, §3.2 |
| Anti-forgery tokens emitted | **0** | 8 | 10 | §5.6, §4.7, §3.7 |
| Anti-forgery validated | **0** | 7 of 12 POSTs | 8 of 13 POSTs | §5.6, §4.7, §3.7 |
| Sign-out verb | **`GET`** | token-protected `POST` | token-protected `POST` | §5.4, §4.4, §3.4 |
| State-changing `GET` (`AddToCart`) | yes | yes | yes | §6.1 |
| Plaintext administrator credential committed | n/a — no provisioning | **yes** | **yes** | §5.5, §4.5, §3.5 |
| Administrator role ever created | **no** | yes | yes | §5.2, §4.5, §3.5 |
| Provisioning idempotent per operation | n/a | **yes** | **no** | §4.6, §3.6 |
| Over-posting control at checkout | **exclude list — permits `Total`** | include list | include list | §5.7, §6.4 |
| Output encoding of the echoed album title | `Server.HtmlEncode` | **none** | **none** | §5.9, §6.2 |
| Raw exception placed on a response channel | no | **yes — one site** | no | §4.9 |
| Transport protection (HTTPS, HSTS) | none | none | none | §6.5 |
| Security response headers | none | none | none | §6.5 |
| `<machineKey>` / data-protection keys | none | none | none | §6.6 |
| Destructive schema initializer | yes | yes | yes | §6.7 |
| Audit trail / application logging | **none** | **none** | **none** | §6.8 |
| Credential-shaped database committed to source control | **yes — an unreferenced tutorial artifact.** No tracked configuration names it, and **no account contents are inferred from it**; the edition's effective store is the host's and is unknown from the repository (§5.1, §6.9) | **yes — the edition's own store**, with its password material intact | **yes — the edition's own store**, with its password material intact | §6.9, §5.1 |
| Live `customErrors` element | no | no | no | §6.10 |
| Scaffolded-but-disabled external login | none | yes — **7** packages | yes — **4** packages | §6.11 |

### 2.3 The distribution is the finding

Read as a whole, the table says something more useful than "a 2013 tutorial application has security
problems". This codebase is **careful in specific, checkable places** — all eight of MVC 5's account
POST actions validate their token [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:55], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:88],
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:113], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:147], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:199], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:236], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:264], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:301]; the checkout confirmation page verifies that the
signed-in user owns the order it is about to display
[src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:71-73]; the post-sign-in redirect is
guarded against open redirection by `Url.IsLocalUrl`
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:384]; three of the four administration
`Albums.Find` calls guard their result and return `HttpNotFound()`
[src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:33-35], [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:74-76], [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:106-108]; and the
cart removal query is correctly scoped to the caller's own cart
[src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:64-66]. Section 7 collects these.

It is **alarming in equally specific places**: a credential in cleartext in two editions, a mutating
`GET` in all three, five unprotected state-changing POSTs in each of the two newer editions, no audit
trail anywhere, and a credential-shaped database committed to source control in each of the three
edition trees — MVC 4's and MVC 5's own credential stores with their password material intact, and in
MVC 3's tree a tutorial membership database that no tracked configuration references (§6.9).

That mixture is what makes the register credible, and it is what a reader needs in order to sequence
the work. A uniformly bad codebase would justify a rewrite; this one justifies a port plus a named
list of controls to add, which is what deliverables 03 and 05 build.

---

## 3. MVC 5 — ASP.NET Identity 1.0 over OWIN cookie authentication

*Every finding in this section holds in **MVC 5** unless its Editions line says otherwise. This is
the edition the port is sourced from, so it is assessed first and in the most depth.*

### 3.1 Authentication posture

**Editions: MVC 5.**

Authentication is entirely OWIN middleware. ASP.NET's own authentication is switched off —
`<authentication mode="None"/>` [src/MVC5/MvcMusicStore/Web.config:32] — and the Forms module is
explicitly removed from the pipeline [src/MVC5/MvcMusicStore/Web.config:38], so nothing in
`system.web` participates. The single enabled middleware is cookie authentication, registered in
`ConfigureAuth` [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:14-18], with a second cookie
registered to hold third-party sign-in state temporarily
[src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:20]. The OWIN entry point that runs it is declared
by assembly attribute — `[assembly: OwinStartupAttribute(typeof(MvcMusicStore.Startup))]`
[src/MVC5/MvcMusicStore/Startup.cs:4] — and calls `ConfigureAuth` then `ConfigureApp`
[src/MVC5/MvcMusicStore/Startup.cs:11-13].

The credential store is ASP.NET Identity **1.0**, EF-mapped and owned by the application:
`ApplicationDbContext : IdentityDbContext<ApplicationUser>` [src/MVC5/MvcMusicStore/Models/IdentityModels.cs:10]
bound to `DefaultConnection` [src/MVC5/MvcMusicStore/Models/IdentityModels.cs:13]. Deliverable 01 §8.1
owns the architecture; the security consequence of the *version* is §3.3 below, because Identity 1.0
is the generation that predates lockout.

### 3.2 Authorization posture, and its enforcement points

**Editions: MVC 5** (MVC 4 shares MVC 5's five and adds a sixth — §4.2; MVC 3 differs materially —
§5.2).

**The count is per edition, and the three editions do not agree on it.** An earlier form of this
section published "five enforcement points" as a property of the application, which was MVC 5's figure
read across all three; MVC 4 has **six** and MVC 3 has **six** with a materially different composition.
The per-edition figures and the rule that produces them are stated here once, and §4.2 and §5.2 then
carry the two other editions' own enumerations rather than a delta from this one.

**The counting rule, stated because a count with no rule is not checkable.** An enforcement point is a
place in **the edition's own source** where an authorization decision is made about the caller's right
to the resource. Three forms qualify and the boundary of each is stated:

- an **authorization attribute** — one application of `[Authorize]` or `[Authorize(Roles = …)]`, at
  class or action level. `[AllowAnonymous]` is an **opt-out** and is never counted; nor is a global
  filter, because **no edition registers an authorization filter globally and none appears in a
  view** — the two commands that establish it, over every `App_Start` folder, every `Global.asax.cs`,
  MVC 4's `Filters` folder and all three `Views` trees, each return `0` and are in §9;
- an **imperative in-action check** — a comparison of a **client-supplied** subject against the
  authenticated principal, whose removal would let one authenticated user act on another's resource;
- a **data-scope predicate** on a mutation — a query predicate that binds the operation to the caller's
  own rows against a client-supplied row identifier. It is counted **once per control**, not once per
  query site: MVC 5's cart predicate appears at seven sites
  [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:38], [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:65], [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:89], [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:100], [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:107], [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:120], [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:186] and
  is one control, evidenced at the removal site because that is the one constraining a mutation.

What the rule deliberately does **not** count is a framework call that scopes an operation by taking
the caller's own identifier as its subject. That is a property of the call rather than a decision in
this source, and the distinction is exactly what separates MVC 5's five from MVC 4's six — §4.2 shows
the pair.

MVC 5 has **five**, and they are not of one kind: **three controller-level attributes plus two
in-action or data-scope checks** — a `3 + 2` split, stated because "five controller-level attributes"
would be wrong on both halves of the count. Only the first three are declarative; the last two are
imperative code inside a method and a query predicate respectively, which is why §7 credits them separately
and why the port cannot carry them forward by moving an attribute:

| Enforcement point | Effect |
| --- | --- |
| `[Authorize]` on `AccountController` [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:15] | Authenticated by default, with `[AllowAnonymous]` opening the sign-in, registration and external-callback actions individually — eight occurrences at [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:44], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:54], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:78], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:87], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:198], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:208], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:263], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:310] |
| `[Authorize]` on `CheckoutController` [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:8] | The whole checkout requires authentication |
| `[Authorize(Roles = "Administrator")]` on `StoreManagerController` [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:12] | The entire administration surface requires the role |
| Ownership check inside `Checkout.Complete` [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:71-73] | The order confirmation is shown only if `o.Username == User.Identity.Name` |
| Cart scoping inside `ShoppingCart.RemoveFromCart` [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:64-66] | The removal targets only a row whose `CartId` matches the caller's cart key |

`StoreController`, `HomeController` and `ShoppingCartController` carry no authorization attribute at
all, which is correct for a public catalogue and an anonymous cart. The two data-scoping checks in the
last two rows are genuine controls and are credited in section 7 — but note the asymmetry recorded in
§6.2: the removal *mutation* is scoped, and the album-title *read* that accompanies it is not.

**The three editions' counts, so that no reader takes MVC 5's five as the application's figure.** Each
is enumerated in full in the section named, under the rule above, and each row of this table is a count
over that enumeration:

| Edition | Enforcement points | Composition | Enumerated in |
| --- | --- | --- | --- |
| **MVC 5** | **5** | 3 controller-level attributes + 1 in-action ownership check + 1 data-scope predicate | §3.2, the table above |
| **MVC 4** | **6** | the same 5 at the same shapes, **plus** a second in-action ownership check — `Disassociate`'s owner comparison [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:126] — which is the only resource-level authorization check on an *account-management* action in the whole repository | §4.2 |
| **MVC 3** | **6** | **4 attributes + 1 in-action ownership check + 1 data-scope predicate**, and the four attributes are placed differently: **no class-level `[Authorize]` on the account controller at all** (F-09-19), two **action-level** ones on `ChangePassword` GET and POST, and one of the four — the administration role guard — **unsatisfiable as shipped** because nothing creates the role (F-09-18) | §5.2 |

**MVC 4's sixth point is a real difference and not a counting artifact, and MVC 5's absence of it is
not a gap.** MVC 4's `Disassociate` receives both `provider` and `providerUserId` from the request and
passes them to `OAuthWebSecurity.DeleteAccount(provider, providerUserId)`
[src/MVC4/MvcMusicStore/Controllers/AccountController.cs:134], a call that carries no notion of who is
asking — so without the comparison at [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:126] any authenticated user could unlink another account's
external login. MVC 5's equivalent action needs no such comparison because the caller's own identifier
**is** the subject of the operation:
`UserManager.RemoveLoginAsync(User.Identity.GetUserId(), new UserLoginInfo(loginProvider, providerKey))`
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:117]. Both editions are protected; only one
of them makes the decision in its own source, and only the one that does can lose it to an edit. MVC 3
has no external-login surface at all (§5.10), so the question does not arise there.

**MVC 3's six is not evidence of a stronger posture, and the composition column is why the raw
number must not be read on its own.** Two of MVC 3's four attributes protect one action — the
`ChangePassword` GET/POST pair — where MVC 4 and MVC 5 protect the entire account controller with one
class-level attribute and open individual actions with `[AllowAnonymous]`. So MVC 3 reaches the same
arithmetic with strictly less coverage, which is F-09-19, and a fourth of its declarative guards can
never be satisfied, which is F-09-18. The count is published per edition precisely so that this
distinction is visible rather than averaged away.

**Finding F-09-01 — the declarative model has no defence in depth.** Severity **Low** on its own.
**Editions: MVC 5 and MVC 4**, at five points and six respectively per the table above. Every one of
those points is the *only* thing standing between an anonymous request and the
resource. There is no second check inside any administration action: `StoreManager.Create` goes
straight to `db.Albums.Add(album)` [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:58]
with no re-verification, and `DeleteConfirmed` goes straight to `Albums.Remove`
[src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:119-120]. This is conventional MVC and
is not itself a defect; it is recorded because it means a single attribute removal or a single routing
change silently exposes the whole administration surface, and there is no logging (§6.8) that would
reveal it had happened.

### 3.3 Cookie, password and lockout policy — inherited and undocumented, not absent

**Editions: MVC 5.** (The same *framing* applies to MVC 4 — §4.3 — and cannot be applied at all to
MVC 3, whose policy is a property of the host — §5.3.)

This is the most consequential framing in the document, and getting it backwards produces a wrong
remediation. The cookie middleware is configured with exactly **two** values — an authentication type
and a login path [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:16-17] — plus the external
sign-in cookie [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:20]. Nothing else is set, anywhere. `git grep -in 'machineKey|sessionState|requireSSL|httpCookies' -- '*.config'` returns **0**
(§9).

The policy is therefore not *absent*. It exists, it is in force on every request, and it is whatever
the framework's defaults happen to be:

| Policy | Where it comes from | Stated in the repository? |
| --- | --- | --- |
| Password minimum length and complexity | `UserManager` defaults; no `PasswordValidator` is assigned anywhere in the edition | **no** |
| Lockout threshold and duration | **the sign-in code path performs no lockout at all**, so no threshold or duration is in force; the shipped store's physical columns are a separate, extraction-gated question — see the finding below | **no** |
| Account confirmation requirement | Identity 1.0 default; no confirmation flow is implemented | **no** |
| Cookie lifetime and sliding expiration | `CookieAuthenticationOptions` defaults, unset at [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:14-18] | **no** |
| Cookie `Secure` attribute | Katana default (`SameAsRequest`), which over plain HTTP (§6.5) means **not secure** | **no** |
| Cookie `SameSite` attribute | Not a Katana 2.0.0 option at all | **no** |
| Persistent-cookie duration when "remember me" is used [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:63] | Middleware default | **no** |

**Finding F-09-02 — the entire authentication policy is inherited and undocumented.** Severity
**Medium**. Nobody chose it and nobody reading the repository can see it. Two practical consequences
follow. First, it cannot be reviewed: there is no artifact to approve or reject. Second, it is
**silently version-coupled** — a framework or middleware upgrade changes the effective policy with no
diff in the application, and a port to a different framework changes it wholesale. Deliverable 05
owns the requirement to set each row above explicitly and to label each as *preserved* or
*deliberate hardening*; the finding here is that today there is nothing to compare against.
Technical Specification §6.4 reaches the same conclusion about framework-default policy inheritance
and is a valid secondary cross-reference.

**Which document owns what here, stated once because the two halves are easy to swap.** This document
owns the **current-state posture**: what is configured, what is inherited, what the inherited value
resolves to, and the consequences of that — every row of the table above, with its evidence.
**Deliverable [05 §6.1](05-aspnet-core-migration-approach.md) owns the target values** — password
minimum length and complexity, lockout threshold and duration, confirmation requirement, cookie
lifetime and sliding expiration, `Secure`, `SameSite`, and the persistent-cookie duration — **and the
`preserved`-or-`hardening` label on each.** No target value for any of those rows appears in this
document, deliberately: where a later section needs one it cites 05 §6.1 rather than repeating it, and
a reader who arrives here from another deliverable looking for the value to assert should follow that
citation. **A gate or assertion elsewhere phrased as "the value this deliverable sets" is pointing at
the wrong document** and should read 05 §6.1; this document supplies the *baseline* such a gate is
compared against, never the target.

**Finding F-09-03 — there is no account lockout in the sign-in code path, and the Identity generation
this edition pins predates the feature entirely; what the shipped store's physical columns are is
gated on authoritative extraction rather than asserted here.** Severity
**High**. Sign-in is a single call, `UserManager.FindAsync(model.UserName, model.Password)`
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:60], with no attempt counter, no delay and
no lockout on failure; the failure path adds a message and redisplays the form
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:68], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:73]. That is not an omission in the
application — ASP.NET Identity **1.0** predates lockout entirely. A printable-string inspection of the
shipped credential database finds `PasswordHash` and `SecurityStamp` but **no** occurrence of
`LockoutEnabled`, `LockoutEndDateUtc`, `AccessFailedCount`, `TwoFactorEnabled` or `EmailConfirmed`
in the tracked binary
[src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205.mdf:first UTF-16LE byte offsets at any alignment 0x732E9 and 0x73336 for the two names found, and not found at any alignment for the five sought and absent, in a 3,211,264-byte file]
(§6.9, §9) — the binary locator form §1.4 requires, which is
the evidence form and the artifact's size rather than a bare path, paired with the ASCII and UTF-16 string
probe in §9 that produced the offsets. That is consistent with
the 1.0 schema and with the pinned package version `1.0.0` that deliverable 02 records (§3.1 there).
The probe command is in §9, and its epistemic limit is stated honestly: **string-probing a binary is
evidence, not proof** — it cannot distinguish an absent column from one stored in a form the probe
does not surface. Deliverable 05 must treat an authoritative `sys.columns` query as the gate on the
Identity migration; the security conclusion drawn here is narrower and safe either way, because the
*code path* contains no lockout regardless of what the schema holds. Combined with the absence of
transport protection (§6.5) and of any logging that would reveal an attack in progress (§6.8), an
online password-guessing attack against this edition is unthrottled and unrecorded.

**The requirement this finding closes on: rate limiting, not lockout — because lockout closes only half
of it.** Adding per-account lockout in the target (`Lockout.MaxFailedAccessAttempts` and
`DefaultLockoutTimeSpan`, [05 §6.1](05-aspnet-core-migration-approach.md)) constrains an attacker who
guesses **one** account's password many times. It constrains an attacker who tries **one** password
against many *unknown* account names not at all: every attempt names an account that does not exist, so
there is no counter to increment and nothing to lock, and the request is answered by the same generic
failure path. Two costs are paid per attempt, and both scale with the attacker's rate rather than with
their success:

- **CPU — in the target, and this is the one place the source is cheaper for the wrong reason.** Once the
  unknown-account branch is made to cost what the known-account branch costs — which the target must do,
  because the source's failure to do it is the timing oracle **F-09-38** — every attempt naming an account
  that does not exist buys one deliberate key derivation for the price of one cheap request. The defence
  against enumeration is therefore also an amplifier, and that is an accepted cost of closing F-09-38
  rather than an argument against closing it. The **source** does not pay this cost at all: it answers an
  unknown name without touching the hasher, which is exactly why it leaks. Nothing in this bullet is a
  property §3.4 credits the source with, and the target assertion that closes it is a **timing**
  assertion over a population of attempts — **uniform verification cost across five account
  populations**, which [05 §4.3](05-aspnet-core-migration-approach.md) carries — and not the content
  equivalence of its negative-test coverage assertion
  [05 §6.4](05-aspnet-core-migration-approach.md), which is necessary and, on its own, silent about
  this. §3.4 states the same pair of requirements from the oracle's side.
- **Log volume.** Every one of those attempts is an `AUTH-1002` record at **Warning** severity, carrying a
  `SubjectPseudonym`, under §6.8.1's catalog. That is correct and required — but it means an unthrottled
  guessing run writes to the audit sink at the attacker's chosen rate, and a sink that fills or throttles
  loses the records of everything else happening at the same time.

The closure criterion is therefore stated in terms of **abuse controls**, with lockout as one of three
parts rather than the whole:

| # | Requirement | Why lockout does not cover it |
| --- | --- | --- |
| 1 | **Endpoint rate limiting on the sign-in and registration POSTs**, partitioned on a **trusted** client identifier — the client address as the platform's forwarded-headers trust model establishes it ([06 §10.1.4](06-azure-hosting-recommendations.md)), never a client-supplied header taken at face value — rejecting over-limit requests with **429** and no account-existence signal in the response | It is the only control that acts on an attempt naming an account that does not exist, so it is the only one that bounds the CPU and log cost above |
| 2 | **A global limiter as the backstop**, so a run that spreads itself across endpoints or across many partitions still meets a ceiling, and the ceiling degrades service rather than the host | An attacker who rotates both the account name and the partition key defeats a per-endpoint limit alone |
| 3 | **Per-account lockout**, at the values [05 §6.1](05-aspnet-core-migration-approach.md) sets and labels as hardening | Necessary, and sufficient only for the single-account case. It is listed third deliberately: it is the part most likely to be implemented and mistaken for the whole |

Deliverable [05](05-aspnet-core-migration-approach.md) owns the limiter registration, its partitioning,
its limits and the 429 contract, and must state the values as it states the other policy values. What this
document owns is the criterion: **F-09-03 is not closed by enabling lockout.** It is closed when an
unauthenticated request stream naming accounts that do not exist is bounded and answered with 429. The
test that proves it drives an over-limit burst of failed sign-ins against non-existent account names and
asserts three things: the over-limit responses are 429; they disclose no account-existence signal, in
content or in timing, any more than the in-limit ones do; and the count of `AUTH-1002` records the burst
produces is **bounded by the limit rather than by the number of attempts**, because a request the limiter
rejects never reaches the sign-in handler and so emits no event class of §6.8.1's catalog — the rejection
itself is a limiter and platform metric, and this catalog is deliberately not extended with a
seventeenth class for it.

### 3.4 Identity and account flows

**Editions: MVC 5.**

The account surface is **fifteen action methods across twelve distinct action names**, three of those
names being verb-attributed `GET`/`POST` pairs — `Login`, `Register` and `Manage`
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:45-317]. The three units are
[01 §5.2](01-architecture-overview.md)'s counting basis, which that section states once and publishes the
command for; this document uses it unchanged rather than a phrase like "public actions", which does not
say which of the three it means. The security-relevant properties:

- **The sign-in failure *message* does not distinguish an unknown user from a wrong password.** One
  string covers both [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:68]. Credited in §7 as
  a **message-level** control and nothing wider: identical response *text* does not make the two
  branches indistinguishable, because they do not perform the same *work*. The response *time* does
  distinguish them, which is **F-09-38** below.
- **Sign-out is a token-protected `POST`.** The action carries both `[HttpPost]` and
  `[ValidateAntiForgeryToken]` [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:300-301], and
  its call site is a real form with a token, submitted by script
  [src/MVC5/MvcMusicStore/Views/Shared/_LoginPartial.cshtml:4-6], [src/MVC5/MvcMusicStore/Views/Shared/_LoginPartial.cshtml:12]. This is the correct design
  and MVC 3 does not have it (§5.4).
- **The external-login flow carries its own anti-forgery value.** A dedicated key
  [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:336] is passed to
  `GetExternalLoginInfoAsync` when linking a login to an existing account
  [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:247]. Note the asymmetry: the *callback*
  after an initial external sign-in does not pass it
  [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:211], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:275] — which is the framework's own
  scaffolded shape, and moot as shipped because no provider is enabled (§6.11).
- **Errors surfaced from Identity are strings, not exceptions.** The helper that copies
  `IdentityResult` failures into model state passes the message
  [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:360]. This is the safe overload, and the
  contrast with MVC 4's single use of the exception overload (§4.9) is exact.

**Finding F-09-04 — sign-in issues the authentication cookie before the cart write it depends on, in
one method, with no transaction and no logging.** Severity **Medium**.
`SignInAsync` calls `AuthenticationManager.SignIn`
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:350] and *then* migrates the cart
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:353], and the migration opens a **second,
independent** `DbContext` to do it [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:32],
saves it [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:37] and overwrites the session cart key [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:39]. There is no transaction spanning the
identity write and the catalogue write, and none is possible — the cookie is issued into the HTTP
response, outside any database transaction. If the cart migration throws, the failure is swallowed by
nothing and logged by nothing (§6.8), and whether the already-issued cookie reaches the browser is
not determined by anything in the repository. The security-relevant half of this is the ordering: the
cart is reassigned to a user whose sign-in has not been confirmed complete. Deliverable 05 owns the
target ordering and the idempotence requirement; the finding here is that today the ordering is the
riskier one and nothing observes the failure.

**Finding F-09-38 — the sign-in path is a username-enumeration oracle by response time: an account
name that does not exist is answered without any password verification at all, so it is answered
measurably faster than a name that does exist.** Severity **High**. **Editions: MVC 5.**

The call is one line: `await UserManager.FindAsync(model.UserName, model.Password)`
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:60], and the branch on its result is the next
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:61], with the single shared failure message at [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:68]. `FindAsync(userName, password)` is
`Microsoft.AspNet.Identity.Core` **1.0.0**'s own method [src/MVC5/MvcMusicStore/packages.config:8], and
its shape is look-up-then-verify: it resolves the name through the user store first, and **returns null
immediately when the store yields no user**, so the password argument is never used and the password
hasher is never called. Only a name that exists reaches the key-derivation step.

**How this was established — and what is retained, since the two are not the same.** The call site and the
pin are files in this checkout; the callee's internals are not readable from them, so the callee was **run**
rather than assumed — the second case §1.4 item 5 sanctions. The pinned package was fetched from nuget.org
at the exact version the manifest names, its `lib/net45/Microsoft.AspNet.Identity.Core.dll` loaded, and
`FindAsync(userName, password)` called three times on a `UserManager<TUser>` built over a counting
`IUserStore`/`IUserPasswordStore` and a counting `IPasswordHasher`.

**The harness was outside this checkout and is not committed**, because §1.3 forbids adding a file here, so
this document retains the package identity, the invocation shape, the reproduction procedure and the
transcribed counter values — and **not** the harness source, the compiler and runtime it was built and run
on, the literal commands, or the raw output. Two limits follow, and they are what keep the finding inside
its evidence.

- **The finding does not rest on the counters.** It rests on three things a reader can check without
  running anything: the call at [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:60], the branch and
  the single shared message at [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:61] and [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:68], and the pin at
  [src/MVC5/MvcMusicStore/packages.config:8]. Given those, the structural point stands on its own —
  `FindAsync(userName, password)` is a **look-up-then-verify** method, and a method that has not found a
  user has nothing to verify a password against, so the two `null` answers cannot cost the same work. **The
  timing channel is real and this document does not soften it**; what the counters add is the *precise
  shape* of the asymmetry, and it is that precision, not the finding, that is the transcribed part.
- **The durable form of the measurement is the procedure, not the table.** Reproduction is four steps and
  needs nothing from this repository beyond the pin at [src/MVC5/MvcMusicStore/packages.config:8]: download
  `microsoft.aspnet.identity.core.1.0.0.nupkg`, unpack it, implement the two counting interfaces, and call
  the method once per case below, reading the counters after each call. Any downstream deliverable that
  needs these numbers as evidence — the equalisation work [05](05-aspnet-core-migration-approach.md) owns,
  or a negative test asserting equalised cost — re-runs that procedure and retains the complete output,
  including tool versions, as a build artifact; it does not cite the table below as a measured constant.

| Case | `FindByNameAsync` calls | `GetPasswordHashAsync` calls | `VerifyHashedPassword` calls | Result |
| --- | ---: | ---: | ---: | --- |
| Name does not exist | 1 | **0** | **0** | `null` |
| Name exists, password wrong | 1 | 1 | 1 | `null` |
| Name exists, password correct | 1 | 1 | 1 | the user |

The two `null` rows are the two branches the response *body* does not distinguish and an attacker's
stopwatch does: they differ by one store read and one deliberately expensive key derivation. The message
at [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:68] is identical across them and that is
worth having, but a message is a *content* control and this is a *timing* channel — §7 credits the
message accordingly and no further.

**Consequences, and the reason this is High rather than Medium.** Enumeration is not the whole harm; it
is the multiplier on two findings already in this register. It converts F-09-03's unthrottled guessing
from a blind search over the whole name space into a targeted one over confirmed accounts, and the one
account whose name needs no discovering is the committed administrator of F-09-05. It is also
completely unrecorded — every probe is answered by the same generic path, and there is no logging
construct of any kind (F-09-32), so a full enumeration sweep leaves the deployment with no evidence
that it happened. Note the asymmetry the target restores deliberately: §6.8.1's `AUTH-1002` records the
`AccountNotFound` outcome **server-side**, so the operator regains the distinction the attacker must be
denied. Recording it in the audit sink and withholding it from the response and from the response time
are the same requirement seen from two sides, and neither substitutes for the other.

**The fix is hardening, not preservation, and it must be labelled as such.** Equalising the work
changes behaviour that the source exhibits today, so it is an **approved deliberate
change** rather than a preserved contract, and it belongs in the same set as the password, lockout and
cookie values [05 §6.1](05-aspnet-core-migration-approach.md) states and **labels as hardening**.

**The target mechanism is deliverable 05's and it is specified, so this document cites it rather than
sketching one.** [05 §4.3](05-aspnet-core-migration-approach.md) defines `UniformVerificationCost`, a
fixed-work service whose target is the cost of the most expensive hash format present in the store: it
spends the full cost on the unknown-account path and **tops up the difference** on a path that already
derived, taking "a derivation ran" as an **observed** fact from the hasher decorator rather than
inferring it. Two properties of that design are worth naming here because they are what actually closes
this finding: it equalises across **every** account population rather than the unknown-versus-known pair
this section is stated on — a migrated legacy-format hash is cheaper than a current-format one, so a
design that equalised only the pair would leave a second, quieter oracle between the two formats — and
it is a top-up rather than a second full derivation, so closing the oracle does not double the cost of
every genuine sign-in.

Two things about the assertion are this document's to require, and both are satisfied by 05's coverage
rows as they stand. First, **the assertion has to be a timing assertion over a population of attempts**
and not a content assertion: comparing two response bodies proves nothing here, which is precisely the
mistake this finding exists to prevent. Two of 05's coverage rows answer that requirement between them,
and they answer different halves of it. The negative-test row of
[05 §6.4](05-aspnet-core-migration-approach.md) requires the **content** equivalence — one generic
failure message for an unknown user and for a wrong password alike — and states the timing requirement
alongside it rather than in place of it. The timing assertion proper is the one
[05 §4.3](05-aspnet-core-migration-approach.md) carries: **uniform verification cost across five
account populations**, compared as a distribution per population against a stated tolerance rather than
as a chosen pair of medians. That is the stronger form and the one that matches the mechanism above,
because a pair comparison can be passed by an implementation that still leaves two of the five
populations distinguishable. Second, **the CPU consequence is real** and is handled by
F-09-03's rate limiting rather than by accepting the oracle: once the unknown-account branch does the
same work as the known-account branch, every unauthenticated attempt costs one key derivation, which is
the amplification F-09-03's criterion 1 bounds.

### 3.5 Secret handling — the plaintext administrator credential

**Editions: MVC 5 and MVC 4** (identically; MVC 3 has no provisioning at all — §5.5).

**Finding F-09-05 — a working administrator username and password are committed to source control in
cleartext, and the repository's own documentation republishes them.** Severity **Critical**.

The credential is two `appSettings` entries, `DefaultAdminUsername` and `DefaultAdminPassword`:

```xml
<add key="DefaultAdminUsername" value="Administrator"/>
<add key="DefaultAdminPassword" value="[REDACTED — read the cited line]"/>
```

at [src/MVC5/MvcMusicStore/Web.config:16-17], and **byte-identically** at
[src/MVC4/MvcMusicStore/Web.config:25-26]. **The password value is deliberately not reproduced here** —
§1.3 states why, and the two locators above are the verification path: opening either line shows the
working value in its own file, which is both stronger evidence and one fewer place the credential exists.
The username is `Administrator`, quoted because it is the same literal as the role name and the
authorization check below, and it is not a secret. They are read at application startup by both editions —
MVC 5 at [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:23-24], MVC 4 at
[src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:23-24] — and used to create a real account holding a
real role. This is not a template placeholder that a deployment overrides; nothing in either edition
requires it to be overridden, and the account is created on first run either way.

The value is also published in prose, under a heading that describes exactly what it is:
[src/MVC5/README.md:75-85], with the username and password on [src/MVC5/README.md:78-79] and the `Web.config` snippet
reproduced at [src/MVC5/README.md:83-84]. So the credential is discoverable without reading any code.

Three aggravating facts, each cited independently:

1. **The account is a full administrator.** The role it receives is `Administrator`
   [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:25], which is the exact string guarding the entire
   administration surface [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:12].
2. **The credential is not the only committed secret material — the resulting password hash is
   committed too.** The Identity database that stores it is tracked in the index, and a string probe finds
   both `PasswordHash` and the literal `Administrator` inside it
   [src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205.mdf:first UTF-16LE byte offsets at any alignment 0x732E9 and 0x3700B respectively, in a 3,211,264-byte file]
   (§6.9, §9) — the binary
   locator form §1.4 requires, evidence form and artifact size, with the probe command in §9.
3. **Rotation does not remediate it.** Changing the value in `Web.config` leaves every prior value in
   git history and leaves the committed database's hash intact. This is why deliverable 08 records
   the committed binaries as debt requiring either history rewriting or explicit acceptance, and why
   the remediation here is not "change the password" but "remove the mechanism".

Deliverable 05 owns the replacement — provisioning moves out of application startup to an operator
command, and the credential leaves configuration entirely. **Nothing is changed here.**

**The requirement this finding closes on: the credential is treated as compromised, not merely
superseded.** Removing the mechanism stops the value being *consumed*; it does not stop the value being
*valid*. Because the password is committed, present in git history, republished in
[src/MVC5/README.md:75-85], and hashed into a tracked database binary, any account it provisioned must be
assumed reachable by anyone who has ever had the repository. Six criteria, and all six are properties
of a deployment rather than of the repository, so none of them conflicts with §1.3:

| # | Requirement | Verified by |
| --- | --- | --- |
| 1 | **The provisioned account is disabled, or its password rotated to a value that has never been committed**, before the application is exposed on any network other than a developer's own | The old value no longer verifies against the account's stored hash, established by the non-mutating operator-side check criterion 2 defines rather than by attempting to sign in with it; the new value exists only in the operator's secret channel. Deliverable [05 §5.5.1](05-aspnet-core-migration-approach.md) owns the mechanism — the Identity data migration writes the account's hash **already neutralized**, so the criterion is met by the load itself rather than by a subsequent action — and [05 §10.2](05-aspnet-core-migration-approach.md) property 3 owns the rotation that makes the account usable again |
| 2 | **The committed value authenticates nothing** in **any** environment, and "any" is enumerated here rather than left to the reader: not staging, not a demonstration instance, not a data-migration rehearsal target, **not the interim Windows App Service host running the un-ported application**, and **not the legacy application if a rollback returns it to service**. The ported target is the easiest of the five to satisfy and the least likely to be the one that fails | **A non-mutating, operator-side verification against the store, run per environment — not a failed sign-in.** Three reads and three assertions, none of which touches the sign-in path: resolve the account by name and assert it **was found**, so a misdirected connection string or an unreachable store fails the check loudly instead of passing it; verify the committed value against the stored `PasswordHash` **directly**, through the same hasher the application resolves, and assert `PasswordVerificationResult.Failed`; and where the remediation chosen was *disabling* rather than rotation, assert the disabling property on the record itself. Nothing is incremented, no cookie is issued and no lockout state changes. **Why not a failed sign-in** — the instrument a reader reaches for first — is set out in the note below the table: it is both weaker as evidence and destructive as a repeated operation. Exactly one property cannot be established by a store read, namely that **no authentication cookie is issued**, and that property is owned by [05 §5.5.1](05-aspnet-core-migration-approach.md), which places the unrestricted end-to-end assertion in the suite's **disposable fixture database** and names the coverage row that carries it. The note below this table confines that assertion to the disposable setting and states why it is never run as a per-environment gate against a live store. For the two **legacy** cases — the interim host and the rollback window — the end-to-end assertion cannot speak at all, because it exercises the **ported** application; criterion 6 owns those two and names where each is executed |
| 3 | **Any environment ever stood up from the committed `App_Data` binaries is treated as compromised too**, since the hash they contain corresponds to the committed value | Such an environment is rebuilt rather than repaired, or its Identity store is re-provisioned. Note that the **legacy** environments this assessment describes are in exactly this state as shipped, which is why §8.2's sequencing keeps them off any network carrying real data |
| 4 | **The requirement is discharged before the hosted target holds real data**, not after — it precedes the production data load rather than following the cutover | Deliverable [03](03-modernization-roadmap.md) carries it as a **gate condition on the Identity migration workstream**, not as a runbook step afterwards; the check is that no environment holding real personal data has ever accepted the committed value |
| 5 | **The remediation cannot be undone by the operator who performs it** — neither the provisioning command nor a later rotation may re-establish a published value | [05 §10.2](05-aspnet-core-migration-approach.md) property 3 requires the command to **refuse** a password matching a published value and exit non-zero having changed nothing; the coverage assertion requires it. Without this criterion the other four are satisfiable and then reversible by one paste from the README this finding cites |
| 6 | **The account is neutralized in the *legacy* store too — before any interim exposure and before any rollback.** Criterion 1's mechanism is the Identity data *migration*, which writes the **ported** store; the un-ported application authenticates against the source Identity database named by its own connection string [src/MVC5/MvcMusicStore/Web.config:12] and is completely unaffected by it. So the account is disabled, or its password rotated to a never-committed value, **in that source store**, before the interim Windows host is exposed on any network other than a developer's own — and it must still be neutralized when a rollback puts the legacy application back in front of users. **Accepting a known-compromised administrator for the duration of a rollback is not a permitted response**: the rollback window is precisely when nobody is watching the administration surface, and the surface is guarded by this one role [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:12] | The same **non-mutating, operator-side verification** as criterion 2, run against the **legacy Identity store** — the account resolved from the source `AspNetUsers` table and the committed value verified against its `PasswordHash` through Identity 1.0's own hasher, which is the format that store holds — at two points: as a precondition of the interim step ([06 §5.6](06-azure-hosting-recommendations.md), alongside the migrated database and the demonstrably disabled initializer), and as a step of the rollback itself ([06 §11.5](06-azure-hosting-recommendations.md)). It is a criterion rather than a remark because the source Identity tables are **retained untouched** as [05 §5.5.1](05-aspnet-core-migration-approach.md)'s rollback position, so a rollback *restores* the exposure rather than inheriting a fixed state. The legacy store cannot be locked out by a failed sign-in — Identity 1.0's sign-in path consults no lockout counter at all, which is F-09-03, and that holds whatever the store's physical columns turn out to be — but that removes only one of the two reasons the sign-in is the wrong instrument, and the store-side check is what this criterion requires in both places |

**Why the verification is a store read and not a failed sign-in, stated here because the failed sign-in
is the instrument a reader reaches for first and it was this section's earlier form.** Two reasons, and
either alone is sufficient.

*It is destructive, and destructive in the one place that matters.* The neutralization
[05 §5.5.1](05-aspnet-core-migration-approach.md) specifies writes a **random-value hash**, deliberately
rather than a null hash or a permanent lockout, so an attempt with the published value against a
neutralized account is an ordinary **wrong-password** attempt — not a short-circuited one. The target's
lockout policy [05 §6.1](05-aspnet-core-migration-approach.md) enables lockout as a labelled hardening,
against a source that has none at all (F-09-03), so that attempt **increments `AccessFailedCount`** and a
run of them locks the account for the lockout window. The account in question is the **only** member of
the only role that guards the administration surface
[src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:12]. And the attempts are not one-off:
criterion 2 enumerates five environments, criterion 6 adds two more executions, and criterion 4 makes the
whole set a **gate condition repeated on the Identity migration workstream** — so the count of failed
authentications aimed at the single administrator account is unbounded over the life of the deployment,
and locking out the only administrator is a matter of arithmetic rather than of bad luck. There is a
second-order effect worth naming, because this document has already used the same argument in the other
direction: a lockout the probe induces produces the `AccountLockedOut` outcome of §6.8.1's `AUTH-1002`,
which **names the account in the log stream** — precisely the disclosure
[05 §5.5.1](05-aspnet-core-migration-approach.md) cites when refusing permanent lockout as the
neutralization mechanism. A gate that manufactures the state the design refused to use is the wrong gate.

*It is also weak as evidence, which is the reason that survives even where nothing can be locked out.* A
rejected sign-in is a **negative over a composite path**, and every one of the following satisfies it
while leaving the exposure completely intact: the account is absent from the store the application is
actually pointed at; the connection string names the wrong database; the store is unreachable; the
sign-in path is misconfigured; the request never reached the application at all. The check passes, the
credential remains valid where it lives, and nobody learns anything. The store-side verification asserts
the two things the criterion is actually about — that the account **was found**, and that the published
value **fails against that account's own stored hash** — and it is the same offline, read-only mechanism
[05 §5.5.1](05-aspnet-core-migration-approach.md) already specifies for its pre-load census, which
"issues no sign-in, touches no lockout counter and writes nothing". Criterion 2 asks for it at the
deployment gate for the same reasons that section asks for it during the load.

*Where an HTTP-level assertion is genuinely irreplaceable, it is **confined** rather than bounded.* One
property cannot be established by any store read: that **no authentication cookie is issued**. That
property belongs to [05 §5.5.1](05-aspnet-core-migration-approach.md)'s end-to-end assertion — the one it
places in the suite's disposable fixture database and names a coverage row for — and against a
**disposable store**, a throwaway database populated by the migration and destroyed with the run, it costs
nothing and is exactly right. Run instead as a **per-environment gate against a live store**, it becomes
the failed sign-in the two paragraphs above have just rejected. So it is confined to the disposable setting and endorsed in no other,
and three requirements make the confinement checkable rather than aspirational.

- **No live-store execution authenticates at all.** Every environment gate of criterion 2, every repetition
  under criterion 4, and both legacy-store executions of criterion 6 are the **non-mutating
  store-and-hasher verification**: resolve the account, verify the published value against that account's
  own stored hash through the hasher for that store's format, assert it **fails**, and write nothing. No
  sign-in is issued, no counter is touched, and there is nothing to undo. Nothing here licenses a
  **standing, scheduled or monitored** probe that authenticates with a known-bad credential, in any
  environment.
- **The disposable run asserts the counters rather than repairing them.** `AccessFailedCount` and the
  lockout state are read **before and after** the attempt and asserted against their expected values — the
  probed account's single expected increment, and **unchanged** for every other account — after which the
  store is **destroyed, not reset**. The distinction is the whole difference between an assertion and a
  side effect: the first states what happened, the second is a further write that can itself fail.
- **There is no restore step, and its absence is the correction rather than an omission.** An earlier form
  of this paragraph licensed the live-store run provided `AccessFailedCount` and the lockout state were
  recorded before the attempt and **restored** afterwards. That is not a safe bound on a live store, for
  two independent reasons. The restore is a **race**: a genuine failed attempt landing between the probe
  and the reset is erased by it, so the tidy-up destroys precisely the evidence lockout exists to
  accumulate. And it **writes to the only member of the only role that guards the administration surface**
  [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:12], at a gate that runs when nobody is
  watching that surface. A probe that mutates the account it exists to protect, and can silently clear a
  real attacker's failed attempts while doing so, is not made acceptable by cleaning up after itself.

**What this is not.** It is not "change the password", which aggravating fact 3 already rules out as
remediation — the history and the committed hash are untouched by any rotation. Rotation is what makes
the *account* safe; removing the mechanism is what makes the *application* safe; and the two are separate
requirements because doing only the second leaves a live administrator account whose password is in a
public file.

**And it is specifically not closed by the two steps a reader would expect to close it.** Deliverable
05's provisioning command removes the two `appSettings` keys, and its Identity data migration carries
every account's `PasswordHash` forward so that existing users can still sign in. Both are correct, and
**together they would carry this credential into the hosted target**: the keys were only the value's
*source*, and after the migration the value's *validity* lives in the target's own `AspNetUsers` row. That
is why criterion 1 above names the load as the place the account is neutralized and criterion 5 names the
refusal that stops it being restored — the finding closes on the state of the migrated account, not on
the absence of the configuration keys.

**And there is a third expectation that closes nothing on its own, which is what criterion 6 exists for.**
Everything above acts on the **ported** store. The un-ported application remains a live consumer of the
committed value — it reads the two `appSettings` keys at startup
[src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:21-45] and authenticates against the source Identity
database its own connection string names [src/MVC5/MvcMusicStore/Web.config:12] — so every hour that
application is reachable is an hour criteria 1 to 5 do not cover, whether the reachability comes from the
interim hosting option or from a rollback. Criterion 6 attaches the neutralization to the **legacy** store
and to the two moments at which that store is put back in front of users, and it admits no
"for the duration of the rollback" exception: a rollback is a planned, rehearsed operation, so the single
role guarding the administration surface is neutralized as part of the plan rather than accepted as a
known compromise while attention is elsewhere.

### 3.6 Provisioning is unobservable, and only partly idempotent

**Editions: MVC 5.** (The MVC 4 comparison is §4.6, and MVC 4 is the better of the two.)

**Finding F-09-06 — administrator provisioning runs as `async void`, so its failures are
unobservable.** Severity **Medium**. The method is declared
`private async void CreateAdminUser()` [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:21] and is
called without awaiting from `ConfigureApp` [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:18].
An exception thrown after the first `await` inside an `async void` method is raised on a thread-pool
thread with no continuation to observe it; the caller has already returned, and startup completes
normally. There is no logger anywhere in the repository (§6.8) to record it either. The security
consequence is precise: **a failure to create the administrator, or to grant the role, is silent** —
the application starts, serves traffic, and the administration surface is simply unreachable, with no
signal distinguishing that from a configuration error. It is also the repository's only `async void`.

**Finding F-09-07 — provisioning is idempotent overall but not per operation, so a partial prior run
is never repaired.** Severity **Medium**. The method checks the role and creates it if missing
[src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:31-36], then checks the user
[src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:38-39] — but the role **membership** is added
*inside* the user-creation branch [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:41-43], not
checked independently. So if a previous run created the user and then failed before
`AddToRoleAsync` at [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:43] — which §3.6's first finding
makes a silent and therefore plausible
outcome — every subsequent startup finds the user present, skips the branch entirely, and never adds
the membership. The account exists, cannot administer anything, and no run will ever fix it.

MVC 4 does not have this defect. The contrast is what makes the pair a concrete, in-repository
demonstration of why idempotence must be checked **per operation** rather than overall — and §4.6, where
both halves sit side by side, is where that statement and its evidence live rather than being restated
here.

### 3.7 Anti-forgery — emission is broad, validation is not

**Editions: MVC 4 and MVC 5.** The finding below holds in **both** newer editions, at the **same five
action locations**; the two differ only in their POST totals and in their token *emission* footprint,
which is F-09-12's separate point (§4.7). MVC 3 has no anti-forgery control in either direction, so its
gap is total rather than partial and is F-09-21 (§5.6) rather than an instance of this finding. The MVC 4
half is set out with its own locators in §4.7; this section carries the finding itself.

`@Html.AntiForgeryToken()` appears **10** times across MVC 5's views, in 10 distinct files
(`git grep -o 'Html.AntiForgeryToken' -- 'src/MVC5/*.cshtml' | wc -l` → `10`; §9). Emission is broad.

**Finding F-09-08 — five state-changing POST actions in each of the two newer editions do not validate
the anti-forgery token, and in MVC 5 two of the five render one on the page that nothing checks.**
Severity **High**. **Editions: MVC 4 and MVC 5.**

The denominators differ per edition, so all three are stated together — a ratio quoted without its
edition is the misreading this finding attracts:

| Edition | POST actions | Validate | **Do not validate** | Where the locators are |
| --- | --- | --- | --- | --- |
| MVC 5 | **13** | 8 | **5** | this section |
| MVC 4 | **12** | 7 | **5** — the **same five surfaces**, at the same line numbers | §4.7 |
| MVC 3 | **8** | **0** | **8** | §5.6, as F-09-21 — a total absence, not an instance of F-09-08 |

There are 13 POST actions in MVC 5. Eight validate:

| Protected POST | Token attribute |
| --- | --- |
| `Account.Login` | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:55] |
| `Account.Register` | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:88] |
| `Account.Disassociate` | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:113] |
| `Account.Manage` | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:147] |
| `Account.ExternalLogin` | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:199] |
| `Account.LinkLogin` | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:236] |
| `Account.ExternalLoginConfirmation` | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:264] |
| `Account.LogOff` | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:301] |

Five do not, and each is a state change:

| Unprotected POST | Declaration | What it changes | Token on the page? |
| --- | --- | --- | --- |
| `StoreManager.Create` | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:53-54] | Inserts an album [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:58-59] | **yes** [src/MVC5/MvcMusicStore/Views/StoreManager/Create.cshtml:11] |
| `StoreManager.Edit` | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:86-87] | Full-entity update [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:91-92] | **yes** [src/MVC5/MvcMusicStore/Views/StoreManager/Edit.cshtml:11] |
| `StoreManager.DeleteConfirmed` | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:116-117] | Deletes an album [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:119-121] | no |
| `ShoppingCart.RemoveFromCart` | [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:54-55] | Decrements or removes a cart row [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:65], [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:67] | no — posted by script without a form [src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:17] |
| `Checkout.AddressAndPayment` | [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:25-26] | Writes the order and its details [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:44], [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:48], [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:51] | no |

**The first two rows are the finding in its sharpest form: the token is emitted into the form and the
action does not look at it.** A reader auditing the views would conclude the administration surface is
protected. It is not. Emission is not a control; validation is.

**Methodology note, because a plausible command undercounts this — in every edition, and always by
hiding the same action.** `grep -c '\[HttpPost\]'` reports **2** POST actions in
`StoreManagerController`, not 3 — its third is declared `[HttpPost, ActionName("Delete")]` on a single
line [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:116], which the bracketed literal
does not match. The correct count comes from the unanchored `grep -n 'HttpPost'` (§9). The cost of the
anchored form is identical in shape in all three editions and is stated numerically so nobody repeats
it: anchored, the census returns **7 / 11 / 12** POST actions for MVC 3 / MVC 4 / MVC 5 against the
correct **8 / 12 / 13**, and therefore **4** unprotected actions in each newer edition instead of **5**,
and **7** in MVC 3 instead of **8**. The action it drops is album delete — the most destructive of the
three administration writes — in every edition. §9 carries both forms side by side.

Deliverable 05 owns the target policy, the conversion of the AJAX endpoint's token transport and the
verb change discussed in §6.1. This document records only the present coverage.

### 3.8 Session and cart identity

**Editions: MVC 5, MVC 4 and MVC 3** — the mechanism is identical in all three; see §6.2 for the
cross-edition finding and its per-edition locators.

MVC 5's cart key is held in session under a constant
[src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:19] and resolved by `GetCartId`
[src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:161-180]: a signed-in user's key is their **login
name** [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:167], an anonymous visitor's is a fresh `Guid.NewGuid()` [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:172]. No `<sessionState>` element
is declared in any edition (`git grep -in 'sessionState' -- '*.config'` → `0`; §9), so session is
in-process. Deliverable 11 owns the scaling and affinity consequences; the security consequence is
§6.2 and §6.6.

### 3.9 Connection strings and data-access privilege

**Editions: MVC 5** (MVC 4's are materially worse — §4.8; MVC 3's is a different engine — §5.8).

Both of MVC 5's connection strings authenticate with `Integrated Security=True` and attach a database
file from the application's own directory
[src/MVC5/MvcMusicStore/Web.config:12], [src/MVC5/MvcMusicStore/Web.config:13]. Two security
properties follow, and they pull in opposite directions:

- **In its favour: no credential is stored in the connection string.** The application presents the
  host process identity, so there is no database password to leak from configuration. This is a real
  control, and deliverable 11 owns why it nevertheless cannot survive a move to Azure SQL.
- **Against it: the identity presented is the application pool's, and its rights are whatever the
  host granted.** `AttachDbFilename` requires the process to hold write access to the database files
  in `App_Data`, and the EF initializer the edition registers is
  `DropCreateDatabaseIfModelChanges<MusicStoreEntities>`
  [src/MVC5/MvcMusicStore/Models/SampleData.cs:9], registered twice — at
  [src/MVC5/MvcMusicStore/Global.asax.cs:20] and again at
  [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:16]. An initializer that can drop and recreate a
  schema requires the connecting principal to hold DDL rights over it. §6.7 records the destructive
  behaviour; the privilege observation is that **MVC 5's runtime identity is necessarily a
  schema-owning identity**, which is far in excess of what serving requests needs.

**Finding F-09-09 — the runtime identity holds schema-modifying rights it needs only on first run.**
Severity **Medium** for MVC 5, **High** for MVC 4 (§4.8, where the application also creates
databases and tables). Deliverable 06 owns the separation of a deployment-time DDL principal from a
least-privileged runtime identity; the finding here is the present conflation of the two.

### 3.10 Error disclosure

**Editions: MVC 5.** (MVC 4 has an additional, controller-level issue — §4.9. MVC 3 matches MVC 5
here.)

MVC 5's entire error-handling policy is a single global filter,
`filters.Add(new HandleErrorAttribute())` [src/MVC5/MvcMusicStore/App_Start/FilterConfig.cs:10].
Two observations, and the second corrects an easy misreading.

- **The error view discloses nothing.** `Views/Shared/Error.cshtml` is nine lines
  [src/MVC5/MvcMusicStore/Views/Shared/Error.cshtml:1-9]. It declares
  `@model System.Web.Mvc.HandleErrorInfo` [src/MVC5/MvcMusicStore/Views/Shared/Error.cshtml:1] but renders **no property of that
  model** — only two static headings [src/MVC5/MvcMusicStore/Views/Shared/Error.cshtml:7-8]. There is no exception message, no stack trace and no
  request detail in it. **Naming the type is not disclosing it**, and the reason this view must be
  rewritten during the port is that `HandleErrorInfo` does not exist in ASP.NET Core, which is
  deliverable 12's territory and not a security finding.
- **`HandleErrorAttribute` engages when custom errors are enabled for the request, and no edition
  configures an explicit element — so the effective mode is the framework default, `RemoteOnly`.** No
  live `<customErrors>` element exists anywhere: all **24** occurrences of the element sit inside
  commented example blocks in the six XDT transform files, and **zero** appear in any of the three live
  `Web.config` files (§6.10, §9). Undeclared is **not** the same as `Off`, and the distinction decides
  what a user sees: a **remote** request that raises an unhandled exception **does** get the generic
  `Error.cshtml`, which discloses nothing, while a request originating on the server host itself
  receives the detailed ASP.NET error page instead — complete with exception type, message and stack
  trace. The exposure is therefore scoped to request origin rather than absent, which is why §6.10
  records it as a finding about an **undeclared policy** rather than about a leaking error view.
  Combined with `<compilation debug="true" …/>`
  [src/MVC5/MvcMusicStore/Web.config:33], which the Release transform removes
  [src/MVC5/MvcMusicStore/Web.Release.config:18] but which is what the committed configuration
  carries, the shipped posture leaves error-detail behaviour to a framework default nobody stated.

**Finding F-09-10 — the checkout swallows every exception silently.** Severity **Medium**. The entire
order-writing transaction is wrapped in a bare `catch` with no exception variable
[src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:58-62]. The exception is discarded, the
view is redisplayed with no message, and — with no logger anywhere (§6.8) — nothing records that a
customer's order failed. This is the inverse of a disclosure problem and is just as serious: the
failure is undetectable and unattributable. It holds in **all three editions**; see §6.8 for the
per-edition locators.

### 3.11 PII, retention and encryption

**Editions: MVC 5, MVC 4 and MVC 3** — the entity is materially the same in all three; see §6.8.

MVC 5's `Order` entity carries nine personal-data fields: `FirstName`
[src/MVC5/MvcMusicStore/Models/Order.cs:23], `LastName` [src/MVC5/MvcMusicStore/Models/Order.cs:28], `Address` [src/MVC5/MvcMusicStore/Models/Order.cs:32], `City` [src/MVC5/MvcMusicStore/Models/Order.cs:36], `State`
[src/MVC5/MvcMusicStore/Models/Order.cs:40], `PostalCode` [src/MVC5/MvcMusicStore/Models/Order.cs:45], `Country` [src/MVC5/MvcMusicStore/Models/Order.cs:49], `Phone` [src/MVC5/MvcMusicStore/Models/Order.cs:54] and `Email` [src/MVC5/MvcMusicStore/Models/Order.cs:61], alongside `Username`
[src/MVC5/MvcMusicStore/Models/Order.cs:18] which links the record to an identity. They are persisted as ordinary columns by
`storeDB.SaveChanges()` [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:51]. No encryption,
no hashing, no tokenization and no column-level protection appears anywhere. §6.8 records the absence
of retention and audit controls with its commands, and **§6.8.2 states the six controls that must be in
place before this data is loaded into a hosted target**, with a named approver each.

**One consequence of `Username` being the link is decided rather than left open**, because it reaches
into every log record the target writes: since `Username` is what makes an order's nine fields
attributable to a person, a login name written into an application log reproduces that link in a store
with different access controls and a different retention period. The actor in every application log
record is therefore the pseudonymous Identity `UserId`, not the login name — deliverable 06 §9.2 owns the
policy and §6.8.1 applies it to every event class, with exactly one sanctioned exception, the
provisioning audit record.

One point in the application's favour, because it bounds the exposure: **no payment data is
collected or stored.** The `Order` entity has no card, account or token field, and the checkout's
only gate is a promo-code string comparison
[src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:33-34] against a compile-time constant
[src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:12]. There is no payment integration to assess — a fact deliverable 01 records as an out-of-scope
capability and which materially reduces the regulatory surface here.

### 3.12 Data protection

**Editions: MVC 5, MVC 4 and MVC 3.** See §6.6 — no edition declares `<machineKey>` or any key
material, and the consequence for cookie and token integrity across hosts and instances is recorded
there, together with what the absence does *not* mean.

### 3.13 Auditability

**Editions: MVC 5, MVC 4 and MVC 3.** See §6.8 — there is no logging construct of any kind anywhere
in the repository. For MVC 5 specifically, the combination that matters is F-09-06 (silent
provisioning failure), F-09-10 (silent checkout failure) and the total absence of a log: three
distinct security-relevant events, none of which leaves a trace.

**What replaces it is specified rather than deferred: §6.8.1 is the security-event catalog** — sixteen
event classes, each with a stable identifier, an actor, a target, its outcome values, its severity — per
class, and per **outcome** in the one row where the catalog says so — and
the closed set of fields it may carry. The three MVC 5 events named above map to `PROV-6001`,
`ORDER-5002` and, for the unrecorded guessing of F-09-03, `AUTH-1002` and `AUTH-1003`.

### 3.14 Dependency exposure

**Editions: MVC 5.**

Deliverable 02 owns the inventory and the version-risk posture; three of its findings are security
conclusions and are cross-referenced rather than re-derived. **None of the three rests on an advisory
identifier, a severity or a count**, because [02 §8.2](02-dependency-inventory.md) asserts none and
this document may not invent what its source declines to state (§1.2). What each rests on instead is a
pin and where that pin sits in the code, which is a repository fact and is cited as one.

- **The aged-pin class of [02 §8.1](02-dependency-inventory.md) lands on the authentication path, not
  on a peripheral package.** That class is the 2011–2013 pins with their exact versions, and MVC 5's
  members of it include the OWIN family — `Microsoft.Owin` `2.0.0`
  [src/MVC5/MvcMusicStore/packages.config:16], `Microsoft.Owin.Security.Cookies` `2.0.0`
  [src/MVC5/MvcMusicStore/packages.config:19] and `Owin` `1.0`
  [src/MVC5/MvcMusicStore/packages.config:28] — alongside `Microsoft.AspNet.Identity.Owin` `1.0.0`
  [src/MVC5/MvcMusicStore/packages.config:10]. Those are not incidental packages:
  `Microsoft.Owin.Security.Cookies` is the cookie authentication middleware
  actually registered at [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:14-18], and the Identity/OWIN
  bridge sits behind the sign-in path. The security observation is therefore about **position and age**
  rather than about any published finding against a version: the oldest unserviced code in this edition
  is the code that authenticates users. Severity is deliverable 07's to set, and what a scan says about
  these pins on the day it runs comes from [02 §8.3](02-dependency-inventory.md)'s directed scan or from
  a reader's own dated run of [02 §8.2](02-dependency-inventory.md)'s route — not from this document.
- **Client-side libraries are self-hosted with no integrity or policy control.** Deliverable 02 §8.1
  records that every client library is served from the application's own `Scripts/` and `Content/`
  directories rather than a versioned CDN, and that none is served with a Subresource Integrity
  attribute or under a Content Security Policy — neither is declared anywhere in the repository
  (§6.5 records the header absence with its command).
- **Nothing in the repository records, retains or gates on dependency-audit output** ([02
  §8.2](02-dependency-inventory.md), F-02-23). Three separate facts, and the third is the one that makes
  this a finding. NuGet's restore-time audit reports against the pins a restore resolves, so the
  exposure is *observable*; at the client's default audit level its warnings are **advisory rather than
  fatal**, so a restore that raises them **still exits `0`** and no build fails; and there is no
  dependency-scanning configuration of any kind
  (`git ls-files | grep -E '^\.github/|dependabot|renovate' | wc -l` → `0`; §9) and no build, tooling or
  CI artifact that keeps the output or reads it again. **So the exposure is reviewable only by running
  the route [02 §8.2](02-dependency-inventory.md) documents — deliberately, no committed snapshot of
  that output exists anywhere in this assessment (§1.2)** — and reviewable is not the same as gated: a
  route a reader must choose to run does not fail a build, and nobody is obliged to run it. Deliverable
  03 owns the gate that would change that, and [02 §8.3](02-dependency-inventory.md) states what the
  implementation phase must add for the gate to have something to read.

---

## 4. MVC 4 — SimpleMembership with Forms authentication

*Every finding in this section holds in **MVC 4** unless its Editions line says otherwise. MVC 4 is
not a migration source and is not the executable behavioural baseline (§1.2), but its findings are
evidence about surfaces MVC 5 shares with it, and it carries two findings the other editions do not.*

### 4.1 Authentication posture

**Editions: MVC 4.**

Forms authentication is on, with its own login URL and a 2,880-minute (48-hour) ticket lifetime
[src/MVC4/MvcMusicStore/Web.config:36-37]. The credential store is SimpleMembership, reached through
`DefaultConnection` [src/MVC4/MvcMusicStore/Web.config:12-17], and it is not initialized by
configuration but by a filter attribute applied to the account controller — `[InitializeSimpleMembership]`
[src/MVC4/MvcMusicStore/Controllers/AccountController.cs:17], whose work happens in
`OnActionExecuting` [src/MVC4/MvcMusicStore/Filters/InitializeSimpleMembershipAttribute.cs:18].
Deliverable 01 §8.2 owns the architecture.

**Finding F-09-11 — the sign-in ticket lifetime is 48 hours, stated but not justified.** Severity
**Low**. `timeout="2880"` [src/MVC4/MvcMusicStore/Web.config:37] is the scaffold default rather than
a decision, and it is the *only* authentication policy value either of the Forms-based editions
states explicitly. MVC 3 carries the identical value [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:27].
Recorded because it is long for an application that also has no lockout (§4.3), no transport
protection (§6.5) and no shared key material (§6.6) — a stolen ticket is usable for two days and
nothing revokes it.

### 4.2 Authorization posture

**Editions: MVC 4.**

**MVC 4 has six enforcement points, one more than MVC 5**, under the counting rule of §3.2. The first
five are identical in shape to MVC 5's — the same three controller-level attributes,
`[Authorize]` on the account controller [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:16],
`[Authorize]` on checkout [src/MVC4/MvcMusicStore/Controllers/CheckoutController.cs:8] and
`[Authorize(Roles = "Administrator")]` on the administration surface
[src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:12], and the same two in-action
data-scoping checks, the order-ownership check on the confirmation page
[src/MVC4/MvcMusicStore/Controllers/CheckoutController.cs:71-73] and the cart-scoped removal
[src/MVC4/MvcMusicStore/Models/ShoppingCart.cs:64-66].

**The sixth has no MVC 5 counterpart and was omitted from an earlier form of this assessment, which
published MVC 5's five as the figure for both editions.** It is a resource-level ownership check inside
the `Disassociate` action, and it is the **only** authorization decision in the entire repository that
guards an *account-management* operation rather than a checkout or a cart:

| Enforcement point | Effect |
| --- | --- |
| Owner comparison inside `Account.Disassociate` [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:126] | The external login is unlinked only if `ownerAccount == User.Identity.Name` — the account name resolved from the submitted provider pair must be the caller's own |

The action is `[HttpPost]` with `[ValidateAntiForgeryToken]`
[src/MVC4/MvcMusicStore/Controllers/AccountController.cs:118-119] and takes both `provider` and
`providerUserId` from the request [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:120]. Its own comment states the intent — *"Only disassociate the
account if the currently logged in user is the owner"* [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:125] — and the check is load-bearing rather
than decorative: the name it compares is resolved **from the submitted pair**,
`OAuthWebSecurity.GetUserName(provider, providerUserId)` [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:122], and the deletion it guards,
`OAuthWebSecurity.DeleteAccount(provider, providerUserId)` [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:134], carries no notion of who is asking.
Remove the comparison and any authenticated user unlinks any account's external login. Class-level
`[Authorize]` at [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:16] does not substitute for it: that establishes *authentication*, and this is
*authorization over a resource the caller named* — the distinction §6.2 makes about the cart, in a
second place.

Two things follow, and both are the reason the point is counted rather than mentioned. **MVC 5 is not
missing a control here** — its `Disassociate` binds the caller's own identifier into the framework call
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:117], so there is nothing for an in-source
check to decide, which is why the rule in §3.2 does not count it and why the difference of one is real.
And **MVC 4's check is the fragile form of the same protection**: it is application code that can be
edited away with no compile error and no log (§6.8), whereas MVC 5's is a property of the call. The
port must not lose either — deliverable 05 owns carrying the account-management authorization forward,
and this is the one shape it cannot get by moving an attribute.

The transaction and last-credential guard that follow the check
[src/MVC4/MvcMusicStore/Controllers/AccountController.cs:129-132] are **not** counted as a seventh
point: they prevent a user removing their own final sign-in credential, which is an integrity rule
about the caller's own account rather than an authorization decision about someone else's.

F-09-01 applies to both editions, at five points in MVC 5 and six in MVC 4.

Unlike MVC 3, the guarded role **is** created (§4.5), so MVC 4's administration surface is actually
reachable.

### 4.3 Cookie, session, password and lockout policy

**Editions: MVC 4.**

The same framing as §3.3 applies and is not restated: the policy is **inherited and undocumented**,
with one exception — the 48-hour ticket lifetime of §4.1 is the single value the repository states.
Everything else is a provider default. Specifically:

- **No `<membership>` element and no `<providers>` block appear in MVC 4's `Web.config`**, so
  SimpleMembership's own defaults govern password storage and validation. What the repository *does*
  fix is the store shape, passed as literals to
  `WebSecurity.InitializeDatabaseConnection("DefaultConnection", "UserProfile", "UserId", "UserName", autoCreateTables: true)`
  [src/MVC4/MvcMusicStore/Filters/InitializeSimpleMembershipAttribute.cs:41].
- **There is no lockout in the code path.** Sign-in is a single `WebSecurity.Login` call
  [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:38-39] with no attempt counter and no
  delay; the failure path adds a generic message [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:47]. F-09-03's *conclusion* — unthrottled,
  unrecorded online password guessing — holds here too, for a different reason: not a framework
  generation that predates lockout, but a code path that never consults one.
- **No `<sessionState>` and no `<machineKey>`** — §6.6, with its command.

### 4.4 Identity and account flows

**Editions: MVC 4.**

**Fourteen action methods across eleven distinct action names**, three of those names being
verb-attributed `GET`/`POST` pairs [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:24-330], on
[01 §5.2](01-architecture-overview.md)'s counting basis — one method and one name fewer than MVC 5's
surface (§3.4), with the same three pairs. The security-relevant properties, and MVC 4 does well on three
of them:

- **Sign-out is a token-protected `POST`** [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:66-67],
  posted from a real form with a token [src/MVC4/MvcMusicStore/Views/Shared/_LoginPartial.cshtml:5].
  MVC 3 does not have this (§5.4).
- **The sign-in failure *message* does not distinguish an unknown user from a wrong password** — one
  string for both failure modes [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:47]. This is a
  message-level control only, on the same reading as §3.4's. Whether MVC 4's `WebSecurity.Login`
  [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:38-39] shares MVC 5's timing channel
  (**F-09-38**) is **not established here**: SimpleMembership's verification path was not exercised, and
  this document does not assert a finding it has not evidenced. F-09-38 is therefore scoped to MVC 5,
  and confirming or excluding MVC 4 is an item for the host verification deliverable 10 owns.
- **A provider exception is mapped to a safe message rather than surfaced.**
  `catch (MembershipCreateUserException e)` [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:105]
  passes only `e.StatusCode` through a lookup [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:107]. This is the correct handling — and it makes the
  single site that does the opposite, thirty lines later, unmistakably an oversight rather than a
  house style (§4.9).
- **Registration does disclose one fact deliberately:** a duplicate user name produces a
  field-specific "User name already exists" message
  [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:302]. Severity **Low** — user-name
  enumeration through a registration form is a conventional trade-off against usability, and it is
  noted rather than argued.

### 4.5 Secret handling — the same credential, a different provisioning path

**Editions: MVC 4 and MVC 5.**

F-09-05 is a shared finding and is stated once, in §3.5, with both `Web.config` locations. What is
specific to MVC 4 is the mechanism that consumes it.

Provisioning is synchronous and runs from `Application_Start`: `AppConfig.Configure()` is called at
[src/MVC4/MvcMusicStore/Global.asax.cs:27], which calls `CreateAdminUser()`
[src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:18]. That method reads the two settings
[src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:23-24], primes the membership provider outside a
request by invoking the filter attribute directly
[src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:27], and then provisions through SimpleMembership and
the classic `Roles` static:

| Step | Call | Line |
| --- | --- | --- |
| Create the user if absent | `WebSecurity.UserExists` / `WebSecurity.CreateUserAndAccount` | [src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:29-30] |
| Create the role if absent | `Roles.RoleExists` / `Roles.CreateRole` | [src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:32-33] |
| Add the membership if absent | `Roles.IsUserInRole` / `Roles.AddUserToRole` | [src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:35-36] |

The account created is the one named by the redacted `DefaultAdminUsername` setting
[src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:23], and the role it is granted is the code literal
`Administrator` [src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:25] — which is the exact string
guarding the administration surface
[src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:12]. The role name is a literal in
source and is not redacted; only the credential values are.

### 4.6 MVC 4's provisioning is more idempotent than MVC 5's

**Editions: MVC 4 (favourably) contrasted with MVC 5.**

This is the one place the older edition is unambiguously better, and it is worth recording precisely
because the target design depends on the distinction.

The three steps in the table above are **three independent checks**
[src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:29], [src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:32], [src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:35]. Each guards its own operation. A
prior run that created the user but failed before granting the role is therefore **repaired** on the
next startup: the user check short-circuits, the role check short-circuits, and the membership check
finds the gap and closes it at [src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:36].

MVC 5 nests the membership grant inside the user-creation branch
[src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:41-43] and so never repairs that state — F-09-07.
MVC 5 also runs the whole thing as `async void` [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:21],
so the partial failure that creates the gap is itself silent — F-09-06; MVC 4's is synchronous
[src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:21] and an exception propagates out of
`Application_Start`, failing the application start loudly.

**The pair is in-repository evidence that idempotence must be checked per operation, not overall.**
Deliverable 05 owns that requirement for the replacement operator command; this section is where the
evidence for it lives.

### 4.7 Anti-forgery — the same shape as MVC 5, with a different emission footprint

**Editions: MVC 4.**

`@Html.AntiForgeryToken()` appears **8** times in MVC 4's views
(`git grep -o 'Html.AntiForgeryToken' -- 'src/MVC4/*.cshtml' | wc -l` → `8`; §9). MVC 4 has **12**
POST actions; **7** validate and **5** do not.

**F-09-08 holds here unchanged, and this is not an MVC 5 finding with an MVC 4 analogue.** The finding
is stated in §3.7 and its **Editions** value is *MVC 4, MVC 5*: the five unprotected actions are the
same five surfaces, at the **same line numbers** in both editions, so MVC 4's exposure is identical in
kind and identical in location. Only two things differ — the denominator, **5 of 12** here against
**5 of 13** in MVC 5, and the token *emission* footprint, which is F-09-12 below rather than part of
F-09-08.

The seven protected are all in the account controller
[src/MVC4/MvcMusicStore/Controllers/AccountController.cs:35], [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:67], [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:89], [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:119], [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:163], [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:227],
[src/MVC4/MvcMusicStore/Controllers/AccountController.cs:271]. The five unprotected are the same five surfaces as MVC 5's:

| Unprotected POST | Declaration | What it changes |
| --- | --- | --- |
| `StoreManager.Create` | [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:53-54] | Inserts an album |
| `StoreManager.Edit` | [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:86-87] | Full-entity update [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:91] |
| `StoreManager.DeleteConfirmed` | [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:116-117] | Deletes an album |
| `ShoppingCart.RemoveFromCart` | [src/MVC4/MvcMusicStore/Controllers/ShoppingCartController.cs:54-55] | Decrements or removes a cart row |
| `Checkout.AddressAndPayment` | [src/MVC4/MvcMusicStore/Controllers/CheckoutController.cs:25-26] | Writes the order and its details |

**Finding F-09-12 — one edition difference cuts in MVC 4's favour and is easy to invert.** Severity
**Low** — a defence-in-depth and audit-method finding with no direct exploit path of its own, on the
four-level scale §1.6 defines, and recorded to prevent a wrong conclusion. All eight of MVC 4's emitted tokens are in
account views and the layout partial; **none** is in an administration view. MVC 5 emits two more, and
both of the extras are in the administration views whose actions do not validate them
[src/MVC5/MvcMusicStore/Views/StoreManager/Create.cshtml:11],
[src/MVC5/MvcMusicStore/Views/StoreManager/Edit.cshtml:11]. So MVC 5 has *broader emission* and the
same validation gap, which means MVC 5 is the edition where a view-level audit is actively
misleading. Counting emitted tokens is not a measure of anti-forgery coverage in either edition, and
the higher count belongs to the edition that looks better than it is.

The methodology note in §3.7 applies identically here: MVC 4's `StoreManagerController` also declares
its third POST as `[HttpPost, ActionName("Delete")]` on one line
[src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:116].

### 4.8 The DDL-privileged connection string

**Editions: MVC 4.** (MVC 5's weaker version of this is F-09-09 in §3.9.)

**Finding F-09-13 — MVC 4's runtime identity must be able to create databases and tables, and the
application does both at runtime.** Severity **High**.

Both connection strings authenticate as the host process. `DefaultConnection` targets
`Data Source=(LocalDb)\v11.0` [src/MVC4/MvcMusicStore/Web.config:13] with
`Integrated Security=SSPI` [src/MVC4/MvcMusicStore/Web.config:15] and attaches a file-based MDF
[src/MVC4/MvcMusicStore/Web.config:16]; `MusicStoreEntities` targets `(LocalDB)\v11.0`
[src/MVC4/MvcMusicStore/Web.config:19] with `Integrated Security=True` [src/MVC4/MvcMusicStore/Web.config:21] and attaches its own
[src/MVC4/MvcMusicStore/Web.config:20]. Neither carries a stored password, which is the same point in its favour recorded in §3.9.

What makes MVC 4 worse than MVC 5 is that the privilege requirement is not implied — it is exercised
explicitly, in application code, on the credential store:

- `((IObjectContextAdapter)context).ObjectContext.CreateDatabase()`
  [src/MVC4/MvcMusicStore/Filters/InitializeSimpleMembershipAttribute.cs:37] — the application
  **creates the credential database** if it does not exist [src/MVC4/MvcMusicStore/Filters/InitializeSimpleMembershipAttribute.cs:34].
- `autoCreateTables: true`
  [src/MVC4/MvcMusicStore/Filters/InitializeSimpleMembershipAttribute.cs:41] — the application
  **creates the membership tables**.
- `Database.SetInitializer(new MvcMusicStore.Models.SampleData())`
  [src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:16], where `SampleData` derives from
  `DropCreateDatabaseIfModelChanges<MusicStoreEntities>`
  [src/MVC4/MvcMusicStore/Models/SampleData.cs:9] — the application may **drop and recreate the
  catalogue schema** (§6.7).

So the identity that serves every anonymous catalogue request is the same identity that can create a
database, create the tables that hold password hashes, and drop the schema that holds orders. That is
privilege far in excess of need, and it is granted for a first-run convenience. Deliverable 06 owns
the target separation of a deployment-time DDL principal from a least-privileged runtime identity;
deliverable 11 owns why `Integrated Security` cannot reach Azure SQL as written; deliverable 10 owns
the `(LocalDb)\v11.0` instance-name problem, which is a build-and-run prerequisite rather than a
security finding. **None of them is re-argued here.**

### 4.9 Error disclosure — in the controller, not the error view

**Editions: MVC 4 only.** This is the finding most easily misattributed, so its evidence is set out
in full and the negative half is stated as explicitly as the positive half.

**Where it is not.** `Views/Shared/Error.cshtml` is **11 lines**
[src/MVC4/MvcMusicStore/Views/Shared/Error.cshtml:1-11] and contains
only generic apology text: a title assignment [src/MVC4/MvcMusicStore/Views/Shared/Error.cshtml:1-3], a heading [src/MVC4/MvcMusicStore/Views/Shared/Error.cshtml:5] and a paragraph offering to go
back [src/MVC4/MvcMusicStore/Views/Shared/Error.cshtml:7-11]. It declares **no `@model`**, reads **no exception**, and renders **no** message, type,
stack trace or request detail. It is not the disclosure site, and any finding that places it there is
wrong. (MVC 5's error view is also innocent, for a slightly different reason — §3.10.)

**Where it is.** At [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:211-213] — inside the
POST `Manage` action [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:162-164], in the branch taken when the signed-in account has no local
password [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:194-195]:

```csharp
catch (Exception e)                       // AccountController.cs:211
{
    ModelState.AddModelError("", e);      // AccountController.cs:213
}
```

`ModelState.AddModelError` has two overloads. The `string` overload records a message; the
**`Exception`** overload records the exception object itself. Line :213 uses the exception overload —
and it is **the only use of that overload anywhere in the repository**. Every other
`AddModelError` call, in all three editions, passes a string
(`git grep -n 'AddModelError' -- 'src/**/*.cs'` → 13 sites, 12 of them strings; §9). Thirty lines
earlier, the *sibling* branch of the same action does the right thing: `catch (Exception)` with no
variable, and a generic message [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:179-182],
[src/MVC4/MvcMusicStore/Controllers/AccountController.cs:190]. One action, two branches, two opposite handling decisions.

The action then redisplays the view [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:219];
`Manage.cshtml` selects the set-password partial in exactly this branch
[src/MVC4/MvcMusicStore/Views/Account/Manage.cshtml:14-21]; and that partial renders
`@Html.ValidationSummary()` [src/MVC4/MvcMusicStore/Views/Account/_SetPasswordPartial.cshtml:10]. So
the exception object is placed on a channel that is rendered into the response.

**What the shipped rendering does with it — measured, not assumed, and the measurement labelled for what
it retains.** The finding below **does not depend on this measurement**; the measurement bounds it, which
is why this assessment ran it rather than leaving the consequence to inference. A harness compiled against
**MVC 4's own committed `System.Web.Mvc.dll` 4.0.20710.0** — the exact assembly this edition binds to,
taken from `src/MVC4/MvcMusicStore/packages/Microsoft.AspNet.Mvc.4.0.20710.0/lib/net40/` — constructed the
same model-state entry and rendered a real `Html.ValidationSummary`.

*How this was established, stated precisely per §1.4's fifth convention.* The harness was written, built
and run **outside this checkout**, because §1.3 forbids adding a file to this repository, and it is
therefore **not committed and not re-executable from this document**: what is retained is the assembly
identity above, the reproduction procedure in §9.3, and the transcribed values below — **not** the harness
source, **not** the compiler or runtime version it was built and run against, **not** the literal command
lines, and **not** the raw console output. Two consequences, and neither is a hedge. The table below is a
**transcribed observation of one run**, so it is quoted as what bounded this finding at authoring time and
never as a durable property of the framework; and the finding itself rests entirely on the three static
citations in the paragraph after it — the exception overload at `:213`, the redisplay at `:219` and the
`@Html.ValidationSummary()` in the partial this branch selects — none of which is affected by whether the
run is reproduced. A reader who needs the bound re-established re-runs the procedure in §9.3 against the same
committed assembly; an implementation that needs it as evidence retains that run's complete output as a
build artifact. Results, as observed:

| Probe | Result |
| --- | --- |
| `AddModelError("", exception)` → `ModelError.ErrorMessage` | `""` (length 0); the `Exception` is retained on the error |
| `Html.ValidationSummary()` output | `<div class="validation-summary-errors"><ul><li style="display:none"></li></ul></div>` |
| `Html.ValidationSummary(true)` output | identical |
| Exception text present in either output | **no** |
| Control: `AddModelError("", "MARKER")` → summary output | `<div class="validation-summary-errors"><ul><li>MARKER</li>` + newline + `</ul></div>` — the message **is** rendered, which is what proves the harness exercises the helper rather than a stub |

**Finding F-09-14 — MVC 4 discloses a raw exception: the `catch` at
[src/MVC4/MvcMusicStore/Controllers/AccountController.cs:211-213] places the exception object itself
into model state, which is the channel the validation summary renders to the user.** Severity **Medium**,
and the disclosure is in the **controller**, not in the error view. That is the finding, and nothing
below amends it: the exception object *is* in model state, *is* carried into view rendering by the
redisplay at [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:219], and *is* handed to
`@Html.ValidationSummary()` in the partial this branch selects
[src/MVC4/MvcMusicStore/Views/Account/_SetPasswordPartial.cshtml:10]. One line of the application's own
code put it there, and no line of the application's code keeps it off the page.

**The measurement above qualifies that finding; it does not replace it.** What the harness run establishes
— on the retained-evidence terms stated with it, so as an observation of that run rather than as a durable
property — is what the *shipped* combination renders: MVC 4's own `System.Web.Mvc` `4.0.20710.0` summary helper
reads `ModelError.ErrorMessage`, which the exception overload leaves empty, so with the framework's
default helper the exception's text does not itself appear in the response — the page emits an error
box containing an empty, hidden list item. The disclosure is therefore **latent in the shipped
rendering and actual in any rendering that reads the error differently**, and each of the following
makes it actual without touching :213: a custom or third-party validation summary that reads
`ModelError.Exception`; a helper or filter that serializes model state to JSON for an API or AJAX
consumer; a change in the framework's message-resolution behaviour across a major version; or a port
to a framework whose model-error rendering differs. A port is precisely what deliverable 05 plans.

Two things the qualification is **not**. It is not grounds for lowering the severity, because the
exception reaches the response channel by the application's own decision at a single line and its
non-appearance is a property of one helper's default behaviour rather than of any control the
application exercises. And it is not grounds for relocating the finding to the error view, which the
negative half of this section disposes of.

**Finding F-09-15 — the same line also destroys the exception for the operator, which is the second
half of the same defect.** Severity **Medium**. The user sees an error container with no message and no
explanation of why setting a password failed. The exception is not shown, not rethrown, and — with no
logger anywhere in the repository (§6.8) — not recorded. It is simply gone. An operator investigating
"users cannot set a password" has no evidence to work from.

Two boundaries, stated so this finding is not stretched:

1. **This is not a justification for rewriting MVC 5's error view.** MVC 5's `Error.cshtml` must be
   rewritten because its model type, `System.Web.Mvc.HandleErrorInfo`
   [src/MVC5/MvcMusicStore/Views/Shared/Error.cshtml:1], does not exist in ASP.NET Core — a
   no-successor construct that deliverable 12 owns. It has nothing to do with disclosure, in either
   edition.
2. **This finding is MVC 4's alone.** MVC 5's equivalent helper uses the string overload
   [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:360]; MVC 3 has no exception-overload call
   at all.

### 4.10 PII, data protection and auditability

**Editions: MVC 4** — all three are cross-edition findings and are recorded once. `Order` carries the
same nine personal-data fields, gated by the same restrictive bind list
[src/MVC4/MvcMusicStore/Models/Order.cs:8]; see §6.4 and §6.8. No `<machineKey>` — §6.6. The checkout
swallows every exception in a bare `catch` [src/MVC4/MvcMusicStore/Controllers/CheckoutController.cs:58]
— §6.8. No logging construct of any kind — §6.8.

### 4.11 Dependency exposure

**Editions: MVC 4.**

Deliverable 02 owns the inventory; two of its findings are security conclusions here.

- **Six DotNetOpenAuth packages, pinned at `4.0.3.12153`, ship to serve four commented-out
  registrations** (deliverable 02 F-02-08, §3.2.3 there; registrations at
  [src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs:17-29]). **Seven, counted as §6.11 counts them**: the
  sixth-and-seventh distinction matters because `Microsoft.AspNet.WebPages.OAuth`
  [src/MVC4/MvcMusicStore/packages.config:23] ships the `OAuthWebSecurity` API those registrations actually
  call [src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs:5], so MVC 4's external-authentication footprint is
  seven packages and not six. §6.11 records the general finding and enumerates all eleven across the two
  editions; the MVC 4 instance is the largest, because an entire OAuth and OpenID relying-party stack plus
  the external-login API is deployed for a disabled feature.
- **MVC 4's aged pins are client-side and serialization packages** — the 2011–2013 class
  [02 §8.1](02-dependency-inventory.md) defines, which in this edition is `jQuery` `1.7.1.1`
  [src/MVC4/MvcMusicStore/packages.config:10], `jQuery.UI.Combined` `1.8.20.1`
  [src/MVC4/MvcMusicStore/packages.config:11], `jQuery.Validation` `1.9.0.1`
  [src/MVC4/MvcMusicStore/packages.config:12] and `Newtonsoft.Json` `4.5.6`
  [src/MVC4/MvcMusicStore/packages.config:30]. All are self-hosted with no Subresource Integrity
  attribute and under no Content Security Policy (§6.5). No advisory identifier or severity is stated
  for them here or in the source, per §1.2; the pins and their vintage are the finding.
- **Four Web API packages serve a mapped route with zero `ApiController` implementations**
  (deliverable 02 F-02-07). The route itself is live —
  `api/{controller}/{id}` [src/MVC4/MvcMusicStore/App_Start/WebApiConfig.cs:12-16], registered at
  [src/MVC4/MvcMusicStore/Global.asax.cs:21]. The security observation is narrow and honest: with no
  controller implementing it, the route matches nothing and serves 404, so it is **deployed surface
  with no reachable handler** rather than an exposed API. It is recorded because it is surface that
  no one is watching, and because any future `ApiController` added to this edition is
  instantly routable with no authorization attribute anywhere in that pipeline.

---

## 5. MVC 3 — classic ASP.NET Membership and Roles, resolved from the host

*Every finding in this section holds in **MVC 3** unless its Editions line says otherwise. MVC 3 is
the oldest and least complete edition, it is neither a migration source nor the executable behavioural
baseline (§1.2), and it is the one whose posture the repository **cannot fully determine** — §5.3 is the
boundary of what can be asserted from the checkout.*

### 5.1 Authentication posture, and the configuration that is not in the repository

**Editions: MVC 3.**

Forms authentication is on, with its own login URL and the same 48-hour ticket lifetime as MVC 4
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:26-28]. Role management is switched on
with a bare element carrying no configuration at all,
`<roleManager enabled="true" />` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:15].
Sign-in goes through the classic statics: `Membership.ValidateUser`
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:41] then
`FormsAuthentication.SetAuthCookie`
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:45].

**Finding F-09-16 — MVC 3 declares no credential store, so its identity configuration is inherited
from the machine and cannot be assessed from this repository.** Severity **High** — not because an
inherited default is necessarily weak, but because an unknown security configuration in a deployed
application is itself the risk.

The evidence is an absence, so it is stated as one and given its command (§9). MVC 3's `web.config`
contains **no `<membership>` element, no `<providers>` block under `roleManager`, and no
`LocalSqlServer` connection string**. Its only connection string is the SQL Server Compact catalogue
entry [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:55-59], which is not a membership
store. Both `Membership` and `Roles` therefore resolve to the **machine-level** ASP.NET SQL providers
declared in the host's `machine.config`, against the host's own connection-string setting.

What that means for this assessment, said exactly:

- **The password policy is a property of the host, not of the repository.** Minimum length, required
  non-alphanumeric characters, hashing algorithm, salt handling, `enablePasswordRetrieval`,
  `enablePasswordReset`, `requiresQuestionAndAnswer`, `maxInvalidPasswordAttempts` and
  `passwordAttemptWindow` are all provider attributes, and **none of them is declared here**.
- **Lockout may or may not exist.** Classic ASP.NET Membership *does* support lockout through
  `maxInvalidPasswordAttempts`, unlike Identity 1.0 (§3.3) — so MVC 3 is the only edition where
  lockout is even possible. Whether it is in force is unknowable from the checkout.
- **This cannot be resolved by reading harder.** It requires inspecting the machine-level provider
  and connection-string settings on the supported Windows runtime. Deliverable 10 owns that host
  verification, and deliverable 01 §8.3 records the same architectural fact. **No password or lockout
  claim about MVC 3 is made anywhere in this document**, and any downstream deliverable that makes
  one without the host check is asserting something no one has observed.

**Finding F-09-17 — every account MVC 3 registers is created with the same two hard-coded
password-recovery literals: one fixed question and one fixed answer.** Severity **High**, conditional
on the host's provider settings — and the condition is worth spelling out rather than hiding behind.
**Both values are withheld here under the redaction rule of §1.3**, because they are recovery
credentials for every account the application creates; the argument positions and the locator are
retained, so the finding is confirmed from the checkout in a single read.

Registration calls
`Membership.CreateUser(model.UserName, model.Password, model.Email, "[REDACTED — fixed recovery question]", "[REDACTED — fixed recovery answer]", true, null, out createStatus)`
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:94], inside the
`Register` POST action [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:88] — **the normal registration path, and the only account-creation path the
edition has.** The fourth and fifth arguments are the recovery question and answer, and they are
**compile-time string literals**: no user input reaches either, and no other code path in the edition
supplies a different pair, so every account MVC 3 has ever created carries the identical answer. The
seventh argument, `true`, is `isApproved`, so accounts are auto-approved with no confirmation step.

**What the repository establishes, and what it does not.** Established: the two literals exist, they
are fixed at compile time, and **every** normal registration receives them. **Not established: that
recovery is reachable.** Whether the pair can be used to take over an account depends on the
effective provider's `requiresQuestionAndAnswer`, `enablePasswordReset` and `enablePasswordRetrieval`
settings, and per F-09-16 MVC 3 declares no provider at all — the effective one is inherited from the
host and nobody has read it. If `requiresQuestionAndAnswer` is enabled together with reset or
retrieval, then one shared answer plus a user name is the whole of the recovery challenge for every
account; if reset and retrieval are both disabled, the literals are inert. The severity is High
because the enabling combination is the classic provider's own default posture rather than an unusual
one, and it is recorded as **conditional** because this repository cannot settle what the host has
configured. **No exploit is claimed and none is demonstrated here** — the exposure is
provider-dependent and unproved, which is precisely why it must be checked. It is the single
strongest reason the host verification deliverable 10 owns must actually be performed rather than
assumed.

**The two literals are treated as compromised, not merely as poor sample data — which is precisely
what withholding them from this document does and does not buy.** Redaction keeps them out of one more
committed file. It does nothing to their validity: they are compile-time literals in tracked source and
in git history, so every account this edition has ever created holds a recovery answer known to anyone
who has ever had a copy of the repository. **Redaction is therefore a property of this document and
never a remediation, and the remediation is rotation** — under the host verification deliverable 10
owns, either the effective provider is proven to require no question and answer, or every affected
account's recovery answer is reset to a per-account value and the literals leave the registration path.
Neither is done here, because §1.3 forbids touching the file, and neither could be done by editing this
assessment. The same rule and the same reasoning govern the administrator credential of F-09-05, which
§3.5 states in full and closes on the same terms.

MVC 4 and MVC 5 are not exposed: neither uses a question-and-answer recovery mechanism at all.

### 5.2 Authorization posture — a guarded surface behind a role nothing creates

**Editions: MVC 3.**

**MVC 3 has six enforcement points**, under the counting rule of §3.2 — the same arithmetic as MVC 4
and a different distribution, which is why §3.2 publishes the composition beside every count:

| Enforcement point | Kind | Effect |
| --- | --- | --- |
| `[Authorize]` on `Account.ChangePassword` **GET** [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:116] | attribute, **action-level** | The form requires authentication |
| `[Authorize]` on `Account.ChangePassword` **POST** [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:125] | attribute, **action-level** | The password change requires authentication |
| `[Authorize]` on `CheckoutController` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/CheckoutController.cs:8] | attribute, class-level | The whole checkout requires authentication |
| `[Authorize(Roles = "Administrator")]` on `StoreManagerController` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:12] | attribute, class-level | The administration surface requires a role — **which nothing in the edition creates**, F-09-18 below |
| Ownership check inside `Checkout.Complete` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/CheckoutController.cs:69-71] | in-action | The order confirmation is shown only if `o.Username == User.Identity.Name` |
| Cart scoping inside `ShoppingCart.RemoveFromCart` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:63-65] | data-scope predicate | The removal targets only a row whose `CartId` matches the caller's cart key |

**Two of the six protect one action, and one of the six can never be satisfied — so the arithmetic
overstates the coverage.** The first two rows are the `ChangePassword` pair; MVC 4 and MVC 5 obtain
strictly wider coverage with a single class-level attribute on the whole account controller and
per-action `[AllowAnonymous]` opt-outs, which is F-09-19 below. The fourth row is a guard on a role no
code creates, which is F-09-18. And MVC 3 has **no** resource-level check on any account-management
action — it has no external-login surface to guard (§5.10), so MVC 4's sixth point (§4.2) has no
analogue here either. Deliverable 07 should read this table as a reason not to size MVC 3 by analogy
with the newer editions, and never to treat six as better than five.

**Finding F-09-18 — MVC 3's administration surface is guarded by the `Administrator` role, and no
code in the edition ever creates that role or grants it to anyone.** Severity **High** as an
availability finding, and it is genuinely security-relevant in both directions.

The guard is real: `[Authorize(Roles = "Administrator")]` on the whole controller
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:12]. The
provisioning is absent: MVC 3 has **no `App_Start` folder at all**
(`test -d src/MVC3/MvcMusicStore-Completed/MvcMusicStore/App_Start` → false; §9), and its
`Application_Start` runs only four registrations — the EF initializer
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Global.asax.cs:34], area registration [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Global.asax.cs:36], the
global filter [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Global.asax.cs:38] and routes [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Global.asax.cs:39]. There is no `CreateAdminUser`, no `Roles.CreateRole` call and no
administrator credential in configuration anywhere in the edition.

Two consequences, and both belong in a security assessment:

- **As shipped, the administration surface is unreachable.** Every request to it redirects to the
  login page and, after a successful sign-in, is refused, because no principal can hold a role that
  does not exist. Deliverable 01 §9 marks this capability "unreachable" in its coverage matrix for
  the same reason. Availability of an administrative function is a security property, and this is a
  failure of it.
- **The guard depends on a store the repository does not control (F-09-16).** Because `Roles`
  resolves to the machine-level provider, an `Administrator` role created on the host — by another
  application sharing the same `aspnetdb`, or by an operator — would silently satisfy this guard.
  The edition's administration surface is therefore gated by a role in a **shared, externally
  managed store**, which is a materially different trust boundary from MVC 4's and MVC 5's
  application-owned stores.

**Finding F-09-19 — MVC 3's account controller is allow-by-default, where both newer editions are
deny-by-default.** Severity **Medium**. MVC 3's `AccountController` carries **no class-level
`[Authorize]`** [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:13];
authorization is applied to individual actions instead, and only to `ChangePassword`
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:116], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:125]. MVC 4
[src/MVC4/MvcMusicStore/Controllers/AccountController.cs:16] and MVC 5
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:15] invert this — the class requires
authentication and individual actions opt out with `[AllowAnonymous]`.

The five actions MVC 3 exposes are all ones that *should* be anonymous or are otherwise self-guarding,
so there is no exploit here today. The finding is the posture: on an allow-by-default controller, a
new action is anonymous unless someone remembers to protect it, and a forgotten attribute is an open
endpoint. On a deny-by-default controller the same mistake produces a locked endpoint, which is
noticed immediately. Deliverable 07 should carry this as a reason not to treat MVC 3 as a source of
patterns.

### 5.3 Cookie, session, password and lockout policy — a property of the host

**Editions: MVC 3.**

Stated once, in F-09-16, and deliberately not elaborated: **this document makes no claim about MVC 3's
password or lockout policy**, because the repository does not contain the configuration that
determines it. The two values that *are* in the repository are the login URL and the 48-hour ticket
lifetime [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:26-28]; F-09-11's observation
about the lifetime applies identically. Session is in-process, with no `<sessionState>` declared
(§6.6), and no `<machineKey>` is declared either (§6.6).

### 5.4 Identity and account flows — including a sign-out over `GET`

**Editions: MVC 3.**

**Finding F-09-20 — MVC 3 signs users out over `GET`.** Severity **Medium**. `LogOff` is declared
with **no verb attribute** [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:69]
— the comment directly above it says so explicitly, `// GET: /Account/LogOff` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:67] — and it performs
a state change, `FormsAuthentication.SignOut()` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:71], before redirecting [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:73]. Any third-party page
that causes the browser to issue a `GET` to `/Account/LogOff` — an `<img>` tag is sufficient — signs
the visitor out. The impact is denial of session rather than compromise, which is why this is Medium
and not High; the shape is the same shape as §6.1's `AddToCart`, and §6.1 states why no anti-forgery
policy can cover it.

Both newer editions fixed this: MVC 4 [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:66-67]
and MVC 5 [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:300-301] declare `LogOff` as a
token-protected `POST`. MVC 3 is the outlier.

Three properties of MVC 3's flows are in its favour and are recorded because §5 would otherwise read
as uniformly worse than it is:

- **Its open-redirect guard is the most defensive of the three.** Alongside `Url.IsLocalUrl` it adds
  explicit checks that the return URL is non-trivial, starts with a single slash, and is neither
  `//` nor `/\`
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:46-47]. MVC 5
  relies on `Url.IsLocalUrl` alone [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:384].
- **It migrates the cart *before* issuing the authentication cookie**
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:43], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:45] — the
  safer ordering, and the exact inverse of MVC 5's, which is F-09-04. The same ordering holds on its
  registration path
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:98], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:100].
- **Its failure messages are generic and its exceptions are discarded rather than surfaced.** Sign-in
  failure [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:58],
  registration failure through a status-code lookup [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:105], and change-password
  failure through `catch (Exception)` with no variable and a generic message [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:140-143], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:151]. MVC 3
  has no equivalent of MVC 4's F-09-14.

### 5.5 Secret handling

**Editions: MVC 3.**

No credential of any kind appears in MVC 3's configuration: its `appSettings` block holds three
framework settings and nothing else
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:8-12], and its single connection string
carries no password [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:55-59]. F-09-05 does **not** apply to MVC 3, and this assessment does not
extend it there.

That is a consequence of having no provisioning rather than a decision, and it comes at the cost of
F-09-18. But two hard-coded secret-adjacent literals do exist in the edition: the recovery question
and answer of F-09-17
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:94].

**One credential-shaped binary sits in MVC 3's tree, and what it is must be stated exactly, because
the obvious reading is wrong.** `src/MVC3/MvcMusicStore-Assets/Data/ASPNETDB.MDF` is tracked and does
carry a classic ASP.NET Membership schema: `aspnet_Membership`, `aspnet_Users`, `Password` and
`PasswordSalt` are all present in it
[src/MVC3/MvcMusicStore-Assets/Data/ASPNETDB.MDF:first UTF-16LE byte offsets at any alignment 0x1C4AE, 0xE8F62, 0x5485E and 0x548F4 respectively, in a 10,485,760-byte file]
(§6.9, §9). What the repository
settles about it is narrower than either obvious reading: it is **an unreferenced, credential-shaped
tutorial artifact**, and **nothing in this repository assigns it to the application as a store**. It is
equally not evidence *against* any particular host configuration — which store the edition actually
resolves to is the host's property and is unknown here (F-09-16). Three facts establish the first half:

- **No tracked configuration references it.** `git grep -il 'ASPNETDB' -- 'src/'` returns **0** (§9),
  and MVC 3's `web.config` declares no `LocalSqlServer` connection string and no membership or role
  provider at all (F-09-16) — its only connection string is the SQL Server Compact catalogue entry
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:55-59]. Deliverable 10 §10.1 reaches the
  same conclusion from the build-prerequisite side and owns it.
- **The repository labels the folder it sits in as tutorial material.** `MvcMusicStore-Assets` is
  described as "the assets you will need to build the application, including images, CSS, database
  files" [src/MVC3/readme.txt:11-12], and its own readme says only that "the Data directory contains
  a database (only used if you won't be using SQL Server CE)"
  [src/MVC3/MvcMusicStore-Assets/readme.txt:6] — a statement about the catalogue, with nothing
  designating a credential store for the completed application.
- **MVC 3 provisions no account, so no deployed account material can be inferred from it.** The
  edition has no `App_Start` folder, no `CreateAdminUser`, no `Roles.CreateRole` call and no
  administrator credential in configuration (F-09-18, with its commands in §9). Whatever accounts
  this file happens to contain are the tutorial's, and this assessment infers **no** deployed or
  seeded account, no administrator, and no password policy from it.

So the finding here is narrow and real: a **committed, credential-shaped database carrying a
membership schema with password and salt columns** is tracked in this repository, which is an exposure
worth remediating on its own terms (F-09-34) and a reason the deliverable 08 hygiene entry exists.
What it is **not** is evidence about MVC 3's deployed identity configuration: that remains
**undetermined pending the host verification deliverable 10 owns**, exactly as F-09-16 states.

### 5.6 Anti-forgery — absent in both directions

**Editions: MVC 3.**

**Finding F-09-21 — MVC 3 has no cross-site request forgery control at all: zero tokens emitted and
zero validated.** Severity **High**.

Both halves are absences, so both carry their commands (§9):
`git grep -n 'ValidateAntiForgeryToken' -- 'src/MVC3/' | wc -l` → **0**, and
`git grep -o 'Html.AntiForgeryToken' -- 'src/MVC3/*.cshtml' | wc -l` → **0**.

MVC 3 declares **8** POST actions
(`git grep -n 'HttpPost' -- 'src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/*.cs' | wc -l` → `8`),
every one of them a state change and none of them protected:

| Unprotected POST | Declaration |
| --- | --- |
| `Account.LogOn` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:36-37] |
| `Account.Register` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:87-88] |
| `Account.ChangePassword` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:126-127] |
| `StoreManager.Create` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:48-49] |
| `StoreManager.Edit` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:77-78] |
| `StoreManager.DeleteConfirmed` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:103-104] |
| `ShoppingCart.RemoveFromCart` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/ShoppingCartController.cs:52-53] |
| `Checkout.AddressAndPayment` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/CheckoutController.cs:25-26] |

Sign-in and change-password are the notable entries — a login CSRF and a forced password change are
both live shapes against an unprotected form. The three administration POSTs are unreachable as
shipped for the separate reason in F-09-18, which reduces their practical exposure without repairing
the control. The same methodology caveat as §3.7 applies: `StoreManager`'s third POST is declared
`[HttpPost, ActionName("Delete")]` on one line
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:103].

### 5.7 Over-posting — MVC 3 uses an exclude list where the newer editions use an include list

**Editions: MVC 3 only.** §6.4 records the cross-edition framing; this is the divergence.

**Finding F-09-22 — MVC 3's `Order` entity permits mass assignment of every property except
`OrderId`, and the one that matters is `Total`.** Severity **High**.

All three editions bind the checkout form straight onto the `Order` **entity** through
`TryUpdateModel(order)` — MVC 3 at
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/CheckoutController.cs:29], MVC 4 and
MVC 5 at the identical line of their own copies. The only thing constraining which properties bind is
a class-level attribute on the entity, and **MVC 3's attribute is the opposite kind**:

| Edition | Attribute | Effect |
| --- | --- | --- |
| MVC 3 | `[Bind(Exclude = "OrderId")]` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/Order.cs:8] | **Everything binds except `OrderId`** — including `OrderDate` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/Order.cs:15], `Username` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/Order.cs:18] and `Total` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/Order.cs:63] |
| MVC 4 | `[Bind(Include = "FirstName,LastName,Address,City,State,PostalCode,Country,Phone,Email")]` [src/MVC4/MvcMusicStore/Models/Order.cs:8] | Only those nine bind |
| MVC 5 | the identical include list [src/MVC5/MvcMusicStore/Models/Order.cs:8] | Only those nine bind |

`Username` and `OrderDate` are neutralized on MVC 3's success path, because the action overwrites both
server-side after binding
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/CheckoutController.cs:40-41]. **`Total`
is not overwritten**, and the sequence that follows is where it matters:

1. The order — carrying the client-supplied `Total` — is added and **committed**
   [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/CheckoutController.cs:44-45].
2. Only then is the cart asked to build the order details
   [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/CheckoutController.cs:48-49].
3. Inside that call the correct total is computed and assigned,
   `order.Total = orderTotal` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:153],
   followed by `storeDB.SaveChanges()` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:156].
4. But MVC 3's `ShoppingCart` **owns its own `DbContext`**
   [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:11], which is a *different*
   instance from the controller's [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/CheckoutController.cs:11]
   — and the cart's `GetCart` overload takes no context argument, so no sharing is possible
   [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/CheckoutController.cs:48]. The
   corrective assignment at step 3 lands on an entity tracked by the **controller's** context; the
   save at step 3 commits the **cart's**.
5. The controller redirects without saving again
   [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/CheckoutController.cs:51-52].

Two levels of confidence, kept separate:

- **Certain, from the code alone:** MVC 3's bind list permits `Total` to be supplied by the client,
  and the corrective write is applied to an entity tracked by a context that is not the one the
  following `SaveChanges()` commits. Both are attribute and object-graph facts, and the citations
  above are each a single line.
- **A code-path reading requiring runtime confirmation:** that the client-supplied `Total` therefore
  *persists*. MVC 3's runtime is not available in this environment — deliverable 10 owns that — so
  this assessment does not claim to have observed a mispriced order. It claims that the code contains
  no mechanism that would prevent one, and that the confirmation is a specific, cheap test:
  post the checkout form with an extra `Total` field and read the persisted row.

Deliverable 05 owns the target explicit input model, which removes the whole class of problem for the
migrated edition; deliverable 12 owns the compile break that `TryUpdateModel` becomes. Neither MVC 4
nor MVC 5 is exposed, and this finding must not be generalized to them.

### 5.8 Connection string and provider

**Editions: MVC 3.**

One connection string, and it is not a SQL Server one:
`Data Source=|DataDirectory|MvcMusicStore.sdf` with
`providerName="System.Data.SqlServerCe.4.0"`
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:55-58]. It carries no credential, which is
the same point in its favour as §3.9, and it names a file rather than a server.

The security-relevant observations are two, and both are narrow:

- **The catalogue engine is a retired, out-of-support embedded database.** SQL Server Compact 4.0
  receives no security servicing. Deliverable 02 F-02-12 records it as an undeclared machine-wide
  dependency and deliverable 12 owns it as a no-successor construct; the security consequence is
  simply that an unsupported data engine accumulates unpatched defects.
- **MVC 3 needs a second store for its credentials, and that is the one the repository declares
  nothing about.** The catalogue is SQL Server Compact; the credential store is **not declared at
  all** — the edition carries no membership or role provider element and no `LocalSqlServer`
  connection string (F-09-16) — so which engine, instance and database the classic `Membership` and
  `Roles` APIs resolve to is a property of the host's machine-level configuration and is **not
  determinable from this repository**. This assessment therefore infers **no provider selection, no
  accounts and no password material** for the running edition. Deliverable 10 owns the prerequisite
  and keeps it open as a host-verification item; the security point is that the store holding password
  material is the one the repository says least about — which is also why the committed `ASPNETDB.MDF`
  is described in §5.5 and §6.9 as an **unreferenced, credential-shaped tutorial artifact** rather
  than as this application's store.

The same destructive initializer as the other two editions is registered
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Global.asax.cs:34], against
`DropCreateDatabaseIfModelChanges<MusicStoreEntities>`
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/SampleData.cs:9] — §6.7.

### 5.9 Error disclosure, PII, data protection and auditability

**Editions: MVC 3.**

- **Error disclosure: none, and the error view is innocent.** The global filter is registered inline
  in `Global.asax.cs` rather than an `App_Start` file
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Global.asax.cs:17], and its error view is nine lines
  of generic apology with no `@model` and no exception rendering
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Views/Shared/Error.cshtml:1-9].
  MVC 3 has no equivalent of F-09-14. The
  `customErrors` observation of §6.10 applies: none is declared, and
  `<compilation debug="true" targetFramework="4.0">`
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:16] is what the committed configuration
  carries.
- **The checkout swallows every exception**, in a bare `catch` with no variable
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/CheckoutController.cs:56] — §6.8, and
  in MVC 3 this sits directly on top of F-09-22's partially-committed order.
- **PII: the same nine fields** on the same entity, with the additional exposure of F-09-22 — §6.8.
- **Data protection: no `<machineKey>`** — §6.6.
- **Auditability: none** — §6.8.

**Finding F-09-23 — the newer editions lost the output encoding MVC 3 applies to the echoed album title,
and MVC 3's own encoding is for the wrong context.** Severity **Low**. **Editions: all three** — not MVC 3
alone, despite this section's heading, because the finding has two halves and each holds in a different
edition set: the *loss* of the encoding holds in MVC 4 and MVC 5, and the *wrong-context* encoding holds in
MVC 3. The register row therefore reads `all three`, and the finding is declared here, in the section whose
evidence establishes both halves, rather than split across two sections that would each state half of one
finding.

**Current versus latent, stated because the severity depends on it.** *Currently* there is **no cross-site
scripting vector in any shipped edition**, and this finding does not claim one: the value reaches the page
through jQuery's `.text()` [src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:28], which assigns text
rather than parsing markup, so in every edition as shipped the response is safe whether or not the server
encoded it. What is lost is the **margin**, and the loss becomes *actual* the moment anything downstream
writes that value differently — `.html()`, an `innerHTML` assignment, interpolation into a Razor expression
that is not HTML-encoded, or a client rewrite that renders the JSON payload through a templating library.
Deliverable 05 plans exactly such a rewrite of this view and its script, which is why a Low finding about a
defence-in-depth margin is recorded rather than dismissed. The evidence for both halves follows.

- **One control MVC 3 has and the newer editions lost.** The album title echoed back from the cart
  removal endpoint is HTML-encoded before it enters the response,
  `Server.HtmlEncode(albumName)`
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/ShoppingCartController.cs:68]. This is
  the **only** output-encoding call in the entire repository
  (`git grep -n 'HtmlEncode' -- 'src/**/*.cs' 'src/**/*.cshtml'` → 1 site; §9), and MVC 4
  [src/MVC4/MvcMusicStore/Controllers/ShoppingCartController.cs:75-76] and MVC 5
  [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:75-76] concatenate the value raw.
  Stated precisely, because it is a defence-in-depth point and not an exploit: the value is written
  into the page through jQuery's `.text()`
  [src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:28], which does not interpret markup, so
  there is no cross-site scripting vector in the shipped views of any edition. What was lost is the
  margin — and MVC 3's version is itself imperfect, since HTML-encoding a value destined for a JSON
  payload is encoding for the wrong context. Both directions are what the F-09-23 declaration above
  records, at **Low** and across **all three** editions.

### 5.10 Dependency exposure

**Editions: MVC 3.**

Deliverable 02 owns the inventory. Two security conclusions:

- **MVC 3's MVC framework assembly is not a package.** `System.Web.Mvc, Version=3.0.0.0` is
  referenced with no `HintPath`
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:42], so it resolves from a
  machine-wide install of the out-of-support ASP.NET MVC 3 Tools Update (deliverable 02 F-02-11). The
  security consequence: the framework version actually serving requests is a property of the host and
  is not pinned by this repository, and the product it comes from receives no security servicing.
- **Its three client-side pins are the oldest in the repository** — `jQuery` `1.5.1`
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config:3], `jQuery.Validation` `1.8.0`
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config:5] and `jQuery.UI.Combined` `1.8.11`
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config:6] — the same 2011–2013 class
  [02 §8.1](02-dependency-inventory.md) defines. Self-hosted, no Subresource Integrity, no
  Content Security Policy (§6.5). Age and position are the finding; no advisory identifier or severity
  is asserted, per §1.2.
- **MVC 3 has no external-login surface at all**, so §6.11 does not apply to it — deliverable 01 §8.3
  records the same, with its command.

---

## 6. Cross-edition findings

*Findings are placed here because they are **cross-edition** — established from more than one edition's
evidence rather than from one — and each carries the per-edition locator that proves it. **Read each
finding's own `Editions` line rather than assuming this section means all three**: most hold in all three,
and one, [§6.11](#611-scaffolded-but-disabled-external-login-ships-as-deployed-attack-surface), holds in
MVC 4 and MVC 5 only, because MVC 3 has no external-login surface to disable. Nothing is placed here on
the strength of a single edition's evidence.*

### 6.1 A state-changing action is served over `GET`

**Editions: MVC 3, MVC 4 and MVC 5.**

**Finding F-09-24 — `AddToCart` mutates the database over `GET`, in every edition, and no
anti-forgery policy can cover it.** Severity **High**.

The action is declared with **no verb attribute** in all three editions, and each edition's own
comment says what that means:

| Edition | Declaration | Comment above it | It mutates |
| --- | --- | --- | --- |
| MVC 5 | [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:33] | `// GET: /ShoppingCart/AddToCart/5` [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:31] | album read [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:37-38], cart write [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:43], `SaveChanges()` [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:45] |
| MVC 4 | [src/MVC4/MvcMusicStore/Controllers/ShoppingCartController.cs:33] | `// GET: /ShoppingCart/AddToCart/5` [src/MVC4/MvcMusicStore/Controllers/ShoppingCartController.cs:31] | album read [src/MVC4/MvcMusicStore/Controllers/ShoppingCartController.cs:37-38], cart write [src/MVC4/MvcMusicStore/Controllers/ShoppingCartController.cs:43], `SaveChanges()` [src/MVC4/MvcMusicStore/Controllers/ShoppingCartController.cs:45] |
| MVC 3 | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/ShoppingCartController.cs:33] | `// GET: /Store/AddToCart/5` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/ShoppingCartController.cs:31] | album read [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/ShoppingCartController.cs:37-38], cart write [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/ShoppingCartController.cs:43]; the commit is internal to the cart [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:57] |

The call site is a hyperlink, not a form, in every edition —
[src/MVC5/MvcMusicStore/Views/Store/Details.cshtml:28-29],
[src/MVC4/MvcMusicStore/Views/Store/Details.cshtml:27],
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Views/Store/Details.cshtml:27].

**This cannot be fixed by adding an anti-forgery attribute, and that is the point worth stating
plainly.** Anti-forgery validation requires a token in the request; a `GET` issued by a browser
following a link, an `<img src>`, a prefetch, a crawler or a link-preview fetcher carries no token and
cannot be made to. A state-changing `GET` is therefore outside the reach of *any* CSRF policy — it is
not an unprotected endpoint, it is an unprotectable one. It also means the mutation is triggerable by
mechanisms that are not attacks at all: a search-engine crawler or a browser prefetch will add albums
to carts.

The impact here is bounded — an attacker can add an album to a victim's cart, not remove money from
them — so this is High rather than Critical. It is included at that severity because the *shape* is
what matters: the same shape on a different action is a funds transfer, and MVC 3's `LogOff`
(F-09-20) is the same shape with a different effect.

Deliverable 05 owns the conversion to a token-protected `POST` and the corresponding call-site change,
and records it as an approved interface delta — the path is unchanged, the verb is not, so an existing
bookmark to `/ShoppingCart/AddToCart/5` stops working. **This document changes nothing.**

### 6.2 The cart is identified by an unscoped string, and one read of it is unfiltered

**Editions: MVC 3, MVC 4 and MVC 5.**

Cart ownership is a single string compared in a `where` clause. There is no ownership check beyond it,
no signature and no server-side binding of the key to the session beyond the session entry itself.
The key is held in session under a constant — [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:19],
[src/MVC4/MvcMusicStore/Models/ShoppingCart.cs:19],
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:15] — and resolved by
`GetCartId`: a signed-in user's key is their **login name**
[src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:167], an anonymous visitor's is a fresh
`Guid.NewGuid()` [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:172]. Every cart query filters on that one string —
[src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:37-39], [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:64-66], [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:89], [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:100], [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:107], [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:120],
[src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:186].

**Finding F-09-25 — the cart key doubles as the user's identity, so cart rows are keyed by a
guessable value.** Severity **Medium**. Because a signed-in user's `CartId` *is* their login name
[src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:167], the primary scoping value for another user's cart
is not a secret — it is a user name, which registration will confirm the existence of
[src/MVC4/MvcMusicStore/Controllers/AccountController.cs:302] and which MVC 5 renders in the page
chrome for the signed-in user [src/MVC5/MvcMusicStore/Views/Shared/_LoginPartial.cshtml:10]. Nothing
in the code path lets a request *choose* its own `CartId` — it comes from session, not from the
request — so this is not directly exploitable as shipped, and the severity reflects that. It is
recorded because the design has no margin: any future endpoint that accepts a cart id as a parameter
inherits an authorization bypass, and F-09-26 is exactly that mistake made once already.

**Finding F-09-26 — cart removal is correctly scoped, and the album-title read that accompanies it is
not, so the response discloses a row from any cart.** Severity **Medium**. The precision here is the
finding, and both halves matter:

- **The mutation is scoped.** `RemoveFromCart` filters on both the caller's cart key and the row id
  — [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:64-66],
  [src/MVC4/MvcMusicStore/Models/ShoppingCart.cs:64-66],
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:63-65]. A caller cannot
  delete another user's cart row. This is a real control and is credited in §7.
- **The read is not.** Immediately before that call, the controller looks up the album title for the
  confirmation message using **only** the row id:
  `storeDB.Carts.Single(item => item.RecordId == id).Album.Title` —
  [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:61-62],
  [src/MVC4/MvcMusicStore/Controllers/ShoppingCartController.cs:61-62],
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/ShoppingCartController.cs:59-60]. Any
  caller who can reach the endpoint — it requires no authentication and no anti-forgery token
  (§3.7, §4.7, §5.6) — can enumerate `RecordId` values and read back the album title held in **any**
  user's cart, receiving it in the JSON `Message`
  [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:75-76], [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:83].

Two secondary effects of the same line: `.Single(...)` throws on an unknown `RecordId`, so probing
also produces unhandled exceptions with no log (§6.8); and the disclosed value is not encoded in the
two newer editions (F-09-23).

### 6.3 The administration surface binds the entity directly

**Editions: MVC 3, MVC 4 and MVC 5.**

**Finding F-09-27 — album create and edit bind the `Album` entity itself, with no input model, and
edit performs a full-entity update.** Severity **Medium**.

Both actions take the entity as their parameter — create at
[src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:53-54],
[src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:53-54],
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:48-49]; edit at
[src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:86-87],
[src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:86-87],
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:77-78]. `Album`
carries **no `[Bind]` attribute** in any edition, so every property binds, including the primary key
and both foreign keys.

Edit then marks the whole entity modified —
`db.Entry(album).State = EntityState.Modified`
[src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:91],
[src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:91],
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:82] — which
writes every column from the posted values rather than only the changed ones. An administrator posting
a partial form silently blanks the omitted columns, and a posted `AlbumId` selects which row is
overwritten.

Severity is Medium rather than High because both actions are behind
`[Authorize(Roles = "Administrator")]` (§3.2, §4.2) and the only actor is already a full
administrator — so this is an integrity and least-privilege weakness, not a privilege escalation. It
compounds with F-09-08 and F-09-21: in MVC 4 and MVC 5 the same two actions are also the ones with no
anti-forgery validation, so an off-site page can drive an administrator's browser into a full-entity
overwrite of a row of its choosing. That combination is what raises the pair's practical severity, and
it is why F-09-08 is High.

**Where this finding is resolved, named so the handoff lands.** Deliverable 05 replaces the entity
binding with explicit input models — **`AlbumCreateInputModel`** and **`AlbumEditInputModel`** — mapped
onto a loaded entity so that only the properties the form is entitled to set are written, which closes
both halves at once: the key and foreign keys are no longer bindable, and the update is no longer a
full-entity overwrite. Deliverable 05 owns the models and the mapping; this document records the finding
and the two properties any resolution must have. Emission of `ADMIN-4001`, `ADMIN-4002` and `ADMIN-4003`
from these actions (§6.8.1) is what makes an administration write attributable afterwards, which is the
part F-09-32 leaves absent today.

**Finding F-09-37 — `DeleteConfirmed` is the one administration action that does not null-check its
`Find` result before using it.** Severity **Low**, and recorded here because it belongs to the same
three actions —
[src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:119-120],
[src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:119-120],
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:106-107], where
the other three do (§2.3). An unknown id produces an unhandled exception rather than a 404, and per
F-09-32 nothing logs it — so a probe of the delete endpoint is invisible.

### 6.4 One attribute on one entity is the entire over-posting control at checkout

**Editions: MVC 3, MVC 4 and MVC 5** — with the MVC 3 divergence in §5.7.

All three editions construct an empty `Order` and bind the request onto it —
[src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:28-29],
[src/MVC4/MvcMusicStore/Controllers/CheckoutController.cs:28-29],
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/CheckoutController.cs:28-29]. The entity
holds `OrderId`, `OrderDate`, `Username` and `Total` alongside the nine customer fields
[src/MVC5/MvcMusicStore/Models/Order.cs:12], [src/MVC5/MvcMusicStore/Models/Order.cs:15], [src/MVC5/MvcMusicStore/Models/Order.cs:18], [src/MVC5/MvcMusicStore/Models/Order.cs:64].

**Finding F-09-28 — in MVC 4 and MVC 5, the only thing preventing over-posting into `Username`,
`OrderDate`, `OrderId` and `Total` is a single class-level attribute on the persistence entity.**
Severity **Medium** for MVC 4 and MVC 5 (High for MVC 3, where the attribute is the wrong kind —
F-09-22).

The control is `[Bind(Include = "FirstName,LastName,Address,City,State,PostalCode,Country,Phone,Email")]`
[src/MVC5/MvcMusicStore/Models/Order.cs:8], [src/MVC4/MvcMusicStore/Models/Order.cs:8]. It works, and
it is credited in §7. Four properties of it are worth recording as risk rather than defect:

- **It lives on the entity, not on the operation.** Any other action that ever binds `Order` inherits
  the same nine-property allowance whether that is right for it or not, and any property added to the
  entity is excluded by default — which is the safe direction, but silently.
- **It is invisible at the call site.** `TryUpdateModel(order)`
  [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:29] reads as unrestricted; the restriction
  is in a different file. A reviewer auditing the controller sees no control at all.
- **`Username` and `OrderDate` are belt-and-braces protected**, being overwritten server-side after
  binding [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:40-41]. `Total` is not — it is
  protected by the bind list **alone**, and it is computed later
  [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:151] before the single commit
  [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:51]. MVC 5's ordering — add
  [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:44],
  compute [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:48], commit once [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:51] — is what makes it safe, and MVC 3's different ordering is what
  makes F-09-22 possible. Removing that one attribute in MVC 5 would make the order total
  client-controlled.
- **The promo code bypasses the model entirely.** It is read straight from the raw form,
  `values["PromoCode"]`, and compared against a compile-time constant
  [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:33-34], [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:12] — so it is neither
  validated nor bound, and a value that fails the comparison simply redisplays the view [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:36]. There
  is no rate limit on guessing it and no log of attempts (§6.8). The constant is `"FREE"` and it is
  in the repository; the practical exposure is that the promo mechanism is a shared secret in source
  control, which for this application means checkout cannot complete without it.

Deliverable 05 owns the explicit input model that replaces the attribute — carrying **ten**
properties, the nine the bind list permits plus `PromoCode` — and deliverable 12 owns the fact that
`TryUpdateModel` has no synchronous successor. Neither is re-argued here.

### 6.5 No transport protection and no security response headers

**Editions: MVC 3, MVC 4 and MVC 5.**

**Finding F-09-29 — nothing in any edition requires, redirects to, or asserts HTTPS, and no edition
emits a single security response header.** Severity **High**.

Both halves are absences and both carry commands (§9):

- `git grep -inE 'RequireHttps|requireSSL' -- 'src/**/*.cs' 'src/**/*.cshtml' 'src/**/*.config'` → **0**.
  There is no `[RequireHttps]` filter on any controller or action, no `requireSSL` on any cookie
  configuration, and no HTTPS redirection anywhere.
- `git grep -inE 'Strict-Transport|customHeaders|X-Frame-Options|Content-Security-Policy|X-Content-Type' -- 'src/**/*.config' 'src/**/*.cs'` → **0**.
  No HSTS, no framing policy, no content-type-options, no Content Security Policy, no referrer
  policy.

The development configuration is plain HTTP and the SSL port is empty in every edition that has one —
[src/MVC5/MvcMusicStore/MvcMusicStore.csproj:285] and [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:19],
[src/MVC4/MvcMusicStore/MvcMusicStore.csproj:350] and [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:19], and MVC 3's `<IISUrl>` element is empty
with a plain-HTTP development server port
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:225], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:227-228].

The consequences compound with findings already recorded rather than standing alone. Over plain HTTP:
the authentication cookie's `Secure` attribute resolves to *not secure* (§3.3); MVC 4's and MVC 3's
48-hour Forms tickets (§4.1) travel in clear; credentials submitted to the sign-in forms travel in
clear; and the nine PII fields of the checkout form (§3.11) travel in clear. Without a Content
Security Policy, the self-hosted 2011–2013 client libraries deliverable 02 §8.1 enumerates have no
containment. Without HSTS, a first-request downgrade is available even where TLS is terminated
upstream.

One boundary: a hosting platform can supply TLS termination, HSTS and headers without any application
change, so this finding is about the **application's** posture, not a prediction about a deployment.
Deliverable 06 owns the hosting configuration and deliverable 11 owns the transport gap as a
cloud-readiness item. What is recorded here is that no edition asserts any of it for itself, so
nothing fails closed if the platform does not.

### 6.6 No machine key and no data-protection key material

**Editions: MVC 3, MVC 4 and MVC 5.**

**Finding F-09-30 — no edition declares `<machineKey>` or any other key material, so the keys that
protect authentication tickets, view state and anti-forgery tokens are auto-generated per machine and
differ between machines.** Severity **High** for a multi-instance deployment, **Low** for a single
instance.

The absence is the repository fact, and it reproduces:

```bash
git grep -n 'machineKey' -- '*.config' | grep -v '/packages/'    # no output: zero matches
```

Run against this checkout the command prints nothing, across all fifteen application configuration
files (§9 carries the file count and the case-insensitive form, which also returns 0). Nor is any key
configured in code: the cookie middleware sets only two options
[src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:14-18], and the Forms-based editions configure
nothing beyond the login URL and timeout
[src/MVC4/MvcMusicStore/Web.config:36-37],
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:26-28].

**The mechanism, stated accurately, because the intuitive reading of "no key is configured" is wrong
and produces the wrong remediation.** With the element absent, both keys default to
`AutoGenerate,IsolateApps`: the `AutoGenerate` modifier makes ASP.NET generate a random key **and
store it in the Local Security Authority**, and the `IsolateApps` modifier derives a distinct key per
application from the application ID [Microsoft Learn, *machineKey Element (ASP.NET Settings Schema)*,
<https://learn.microsoft.com/previous-versions/dotnet/netframework-4.0/w8h3skw9(v=vs.100)> — verified
2026-08-28]. So the material is **machine-persistent with per-application isolation**, not
process-ephemeral: an application-pool recycle on the same machine does not by itself invalidate
forms-authentication tickets, view state or anti-forgery tokens, and a restart is therefore not the
failure mode to plan for.

The real limitation is **cross-machine consistency**, and the same primary source states the
requirement directly: the key must be set manually, to a fixed hexadecimal value, to be consistent
across all servers in a web farm. Two consequences follow, and neither is about process lifetime:

- **Authentication tickets and cookies** issued on one machine are rejected on another, because the
  second machine generated different key material. Two servers behind a load balancer, or two App
  Service instances, do not share a signed-in identity unless a fixed `<machineKey>` is configured
  and shared.
- **Anti-forgery tokens** — where they are validated at all (§3.7, §4.7) — are protected by the same
  machine-level material, so a token issued by one machine fails validation on another. Without
  request affinity this turns the eight protected POSTs of MVC 5 into intermittent failures, which is
  the failure mode most likely to be "fixed" by removing the validation.

**MVC 5 differs in one detail that does not change the conclusion.** Its tickets are issued by Katana
cookie middleware rather than by Forms authentication, so the provider that protects them is supplied
by the OWIN host — `Microsoft.Owin.Host.SystemWeb` 2.0.0
[src/MVC5/MvcMusicStore/packages.config:17] — rather than by a `system.web` element. The application
configures no key and no data-protection provider for it either
[src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:14-18], so what protects an MVC 5 ticket is equally
undeclared in the repository and equally unfixed across machines; MVC 5's anti-forgery tokens sit on
ASP.NET's machine-level material regardless, because the token helper does.

A second-order effect belongs here rather than in the bullets, because it is what makes the finding
security-relevant and not merely operational: because the key is generated rather than declared,
nobody can state what protects a ticket, nobody can rotate it deliberately, and nothing in the
repository records its algorithm or length — the same "inherited and undocumented" problem F-09-02
records for authentication policy, applied to key material.

Together with the in-process session that holds the cart key (§3.8, §6.2), the consequence for
scaling out is a cloud-readiness matter that
[11 — Cloud readiness assessment](11-cloud-readiness-assessment.md) owns, and the target key store is
deliverable 06's decision. Neither is re-argued here.

### 6.7 The schema lifecycle is destructive

**Editions: MVC 3, MVC 4 and MVC 5.**

**Finding F-09-31 — every edition registers an initializer that will drop and recreate the catalogue
database, including the orders and personal data in it, whenever the model no longer matches.**
Severity **Critical** as an availability-and-integrity finding.

The class is `SampleData : DropCreateDatabaseIfModelChanges<MusicStoreEntities>` in every edition —
[src/MVC5/MvcMusicStore/Models/SampleData.cs:9],
[src/MVC4/MvcMusicStore/Models/SampleData.cs:9],
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/SampleData.cs:9] — and every edition registers
it at startup: [src/MVC5/MvcMusicStore/Global.asax.cs:20] and again at
[src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:16] (a duplicated registration deliverable 08 owns
as debt; because `SetInitializer` *sets* rather than accumulates, only one initialization runs),
[src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:16], and
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Global.asax.cs:34].

The security framing, as distinct from the operational one: this is **unauthenticated,
unauthorized destruction of customer data triggered by a deployment**, and it requires no attacker.
Any model change — a property added, a type widened — causes the next application start to drop the
database holding every `Order` and its nine PII fields (§3.11) and reseed it from the hardcoded
sample data. There is no confirmation, no backup step, no environment guard and, per §6.8, no log
entry recording that it happened. It also requires the runtime identity to hold DDL rights, which is
F-09-09 and F-09-13.

Deliverable 05 and deliverable 06 own the replacement — deployment-time migrations plus guarded
non-production seeding — and deliverable 08 owns the debt entry. **Nothing is changed here**; the
seeding routine and the initializer registration are left exactly as they are.

### 6.8 PII is stored with no encryption, no retention control and no audit trail

**Editions: MVC 3, MVC 4 and MVC 5.**

**Finding F-09-32 — the repository contains no logging construct of any kind, so no security-relevant
event is recorded anywhere.** Severity **High**.

The absence is total, and it is the strongest absence claim in this document, so it is given the
widest command (§9):
`git grep -inE 'ILogger|log4net|NLog|Serilog|TraceSource|System\.Diagnostics\.Trace|healthMonitoring|EventLog' -- 'src/**/*.cs' 'src/**/*.cshtml' 'src/**/*.config'`
→ **0** matches outside the committed `packages/` payloads. There is no logger, no logging framework,
no trace source, no `<healthMonitoring>` element, no event-log write and no health endpoint. There is
consequently:

- no authentication log — successful and failed sign-ins are indistinguishable and invisible, which
  is what makes F-09-03's unthrottled guessing undetectable as well as unthrottled;
- no authorization-failure log — a rejected administration attempt leaves no trace;
- no administrative-action log — album create, edit and delete are unattributed;
- no order log — and the checkout's bare `catch` discards the only evidence a write failed
  [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:58],
  [src/MVC4/MvcMusicStore/Controllers/CheckoutController.cs:58],
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/CheckoutController.cs:56];
- no provisioning log — which is what makes MVC 5's `async void` failure (F-09-06) silent rather than
  merely asynchronous.

**Finding F-09-33 — personal data is stored in the clear with no retention policy and no deletion
path.** Severity **High**.

The nine fields are `FirstName`, `LastName`, `Address`, `City`, `State`, `PostalCode`, `Country`,
`Phone` and `Email` on `Order` — [src/MVC5/MvcMusicStore/Models/Order.cs:23], [src/MVC5/MvcMusicStore/Models/Order.cs:28], [src/MVC5/MvcMusicStore/Models/Order.cs:32], [src/MVC5/MvcMusicStore/Models/Order.cs:36],
[src/MVC5/MvcMusicStore/Models/Order.cs:40], [src/MVC5/MvcMusicStore/Models/Order.cs:45], [src/MVC5/MvcMusicStore/Models/Order.cs:49], [src/MVC5/MvcMusicStore/Models/Order.cs:54], [src/MVC5/MvcMusicStore/Models/Order.cs:61] — with the same shape in
[src/MVC4/MvcMusicStore/Models/Order.cs:8]'s entity and
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/Order.cs:8]'s. They are written as ordinary
columns [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:51]. No edition contains any
encryption, hashing, tokenization or masking of them; no edition contains a retention period, an
expiry job or a delete-my-data path; and no edition contains an access log that would show who read
them (F-09-32). `Order` has no cascade-delete configuration and there is no account-deletion action
in any edition, so a deleted account's orders — and their PII — persist unreferenced.

The one bound on this, restated from §3.11 because it materially limits severity: **no payment data
is collected or stored** in any edition. There is no card, account or payment-token field on any
entity, and no payment integration exists to assess.

Deliverable 06 owns encryption at rest and key management for the target. The standing retention and
deletion decision is an **approval condition on the exit gate of W9, the domain data migration**, in
deliverable [03](03-modernization-roadmap.md) §5: that gate is not met until the **data owner** has
recorded both a retention period for order and address data and the mechanism by which a deletion
request against them is satisfied, and it closes before the cutover workstream that consumes it
exposes the migrated data. This document records the present state and names no retention period.

#### 6.8.1 The security-event catalog — the requirement that closes F-09-32

**This is the catalog itself, not a requirement to produce one.** Sixteen event classes, each with a
stable identifier, an actor, a target, an outcome, a severity — assigned per class, and per **outcome** in
row 13, the one row whose two outcomes are written at two levels — and the **exact set of fields it is
permitted to carry**. **Ownership follows the producer map of §6.8.1.1, not a single workstream**, because
two of the sixteen classes are emitted by tooling the port does not contain and one is emitted by nothing
this plan builds: deliverable 03 places the
**thirteen** application-produced classes in **W7, the ASP.NET Core port**; the `AUTHZ-3001` records the
**Identity data migration** emits — one per role assignment it loads — in **W8, Identity migration
tooling**; and `PROV-6001` and the `AUTHZ-3001` a grant produces in **W12, the administrator
provisioning tool**. **`AUTHZ-3002` is a reserved identifier with no producer and therefore no
workstream** — the class the note below §6.8.1.1's producer table explains, and the reason the split is
`13 + 2 + 1` rather than `13 + 3`. Verified collection is an exit condition of **W10, hosting and platform
provisioning**, for the thirteen application classes only, because that is where a real sink first exists;
the two tool-produced classes are verified at their own producers' exits, against the separate
destination 06 §9.5.1 gives them, and the reserved class is verified nowhere because nothing emits it.
Deliverable 07 sizes the emission per producer and carries the risk.
**No event class below is optional, and none may be satisfied by a general application log
line.** The reserved class is not an exception to that rule; it is a statement about its producer.
**Nothing emits it, so there is nothing for an implementer to make optional** — and equally, no gate,
test or count in this section may demand that anything does.

**The observability and audit contract is deliverable 06's, and this catalog is its security-side
consumer. That is stated once, here, and is not restated per row.** AAP §0.11.4 assigns the observability
approach to deliverable 06, and this document holds to that without qualification: 06 §9.1 owns the
mechanism — `ILogger` in the application, collected by platform auto-instrumentation, no in-process
telemetry SDK; 06 §9.2 owns the log-privacy policy, which is the never-logged classes, the pseudonymous
actor rule and its two literals, the per-category level pinning, the prohibition on passing a caught
exception object to `ILogger`, the sanitized exception record — whose closed field list is
[06 §9.6.2](06-azure-hosting-recommendations.md)'s shape **R2**, named by shape and **not** reproduced
as a field count here, for the reason stated two paragraphs below — and the order-record field set;
06 §9.3 owns the health endpoints and their sanitized failure description; 06 §9.5 owns the audit
**channel** of record, the producer-to-destination matrix and retention, with
[06 §9.5.1](06-azure-hosting-recommendations.md) owning the provisioning command's audit record — its
field schema, its cardinality and where it durably lands; and 06 §8.4 owns the platform side
of the pseudonym key, delegating only its value shape to §6.8.1's key contract below. **The reserved
security-event log category under which every class below is written is 06's too**: entry 3 of
[06 §9.6.1](06-azure-hosting-recommendations.md)'s register names it and pins its level — for the reason
that register attributes to 06 §9.2 channel 1 — and [06 §9.2.5.1](06-azure-hosting-recommendations.md)
names the unsampled platform channel that carries the records written under it. **This
document therefore names no category literal at all** and cites that entry instead, for the reason that
governs this whole sub-section: a consumer that writes down the selector its own collection depends on is
how two literals come to exist, and if the name ever changes it changes in one place.

What that leaves this catalog is the part 06 does not decide: **which security events must exist, what
each one identifies, and the exact closed set of fields each may carry.** Every field list below is
written against 06 §9.2 and cites it rather than restating it, which is what 06 §9.2 itself asks of this
document. Three consequences follow and are worth making explicit, because a consumer document drifts
precisely where it paraphrases. **Where a field, literal, level or record shape here differs from 06's,
06 governs and this document is wrong** — that is the precedence
[06 §9.6](06-azure-hosting-recommendations.md)'s normative register states over both of its consumers, and
this sentence is this document's acceptance of it rather than a parallel rule; and what this catalog may add
to a record shape is bounded by [06 §9.6.6](06-azure-hosting-recommendations.md), so an addition takes an
amendment to that register rather than a sentence here.

**A field count is a paraphrase, and this document no longer writes one for a shape it does not own.**
The concrete instance is [06 §9.6.2](06-azure-hosting-recommendations.md)'s **R2**, the sanitized
exception record: an earlier form of this document asserted it in two places as a record "of exactly four
fields", which was true of R2 when those sentences were written and stopped being true when 06 amended
the shape at the owning deliverable — R2's sole producer was found to emit request-attribution fields the
closed list did not contain, so the list was widened there. Nothing in this document's own reasoning
depended on the number. What it depends on is that the list is **closed**, and that a message, an inner
message, a `ToString()` and a serialized exception object are outside it whatever its length. Both
sentences therefore name the shape and cite 06 §9.6.2, and neither states how many fields it has. **The
rule generalizes: where this document needs a record shape it names the shape; where it needs a single
field it names that field and cites the owner; it does not count another deliverable's fields** — a count
is the one form of restatement that still looks correct after the owner amends it, and so gives a reader
no signal that it has gone stale.

Third, **this catalog defines no event class for
anything 06 owns**: there is no health-check event class, no telemetry-pipeline class and no retention
class here, because those records are 06 §9.3's and §9.5's and a second definition of them would be a
second contract.

**The ownership split, stated once and in one sentence each, and this paragraph is the recorded
amendment where an earlier form of this sub-section crossed it.** **This document owns the security
finding register `F-09-01`..`F-09-38` (§8), the per-edition posture the register is drawn from, and the
security *requirements* on auditing** — which event classes must exist, what each one must be able to
answer, its identifier, its closed outcome vocabulary, its severity assignment, and its closed
per-class permitted-field set, which is the set [06 §9.6.2](06-azure-hosting-recommendations.md)'s **R5**
delegates here and carries as its `Fields` value. **Deliverable 06 owns the audit sink and the concrete
record schema that satisfies those requirements** — the record envelope and its transport
([06 §9.6.2](06-azure-hosting-recommendations.md) R1 and R5), the log-category register and its level
pinning ([06 §9.6.1](06-azure-hosting-recommendations.md)), the actor domain
([06 §9.6.3](06-azure-hosting-recommendations.md)), the severity domain, the correlation domains, and the
**audit channel of record with its destination and its retention** ([06 §9.5](06-azure-hosting-recommendations.md),
with the per-table retention settings of [06 §9.5.2](06-azure-hosting-recommendations.md)).

**One handoff inside that split is recorded here explicitly, because it is the one an earlier form of
this document crossed in three places at once: the provisioning command's audit record.**
[06 §9.5.1](06-azure-hosting-recommendations.md) owns **that record's field schema, its cardinality — how
many records a run emits on each of its branches — and where it durably lands**, all three. This document
states the security *requirements* on it and nothing else: that the record must exist for every operation
the run reaches, that it must attribute the operation to a validated actor under the precedence of the
fourth convention below rather than to the identity the process happens to hold, that it must name the
account acted on, and that it must never carry the credential, anything derived from a value that failed
validation, or free-form provider text. Earlier forms of row 16, of the branch note below the table and
of the rejection contract's first rule each stated one of the owner's three facts in this document's own
words — a count table, a five-field enumeration and a destination — and all three are **withdrawn in
favour of a citation** rather than annotated, for the reason this whole paragraph exists: a consumer that
writes the owner's value down is a second value, and when the owner's changes the consumer's silently
becomes false.

**The audit channel of record is a log channel and not a database object, which is a change on the
owner's side that this paragraph records rather than paraphrases:** 06 §9.5 **withdrew** the dedicated
audit table, the exporter that would
have moved its rows and the immutable store that would have received them, as artifacts outside the AAP
artifact set of AAP §0.3.1 and §0.4.1. So there is no column, type, width, constraint or grant on 06's side
of this line for either document to own — a record field whose domain the **emitter** enforces is what
stands in place of a column whose domain an engine constraint enforced. Three field sets stated in an
earlier form of this sub-section fell on 06's side of that line and are **withdrawn here rather than
annotated**, because two live copies of one schema are the defect and the second copy is always the one
that goes stale: the **envelope field set** this sub-section used to enumerate — which named a `Timestamp`
application field where R1 states the timestamp is supplied by the collection path and R5 names the durable
fields `OccurredUtc` and `RecordedUtc`; the **single-correlation-identifier** form, superseded by R5's two
domains; and the **durable-representation table**, which restated the owner's column types, widths and
constraint names alongside the producer requirements they satisfy — a restatement that is now doubly
withdrawn, since the table it described no longer exists on the owner's side either. What survives on this
side is stated below as a **requirement on the record and on the channel that carries it** and never as a
description of either, and the further withdrawal already recorded in the conventions below — the actor
literal `unresolved`, which added a third value to a two-literal domain 06 §9.6.3 closes — is an instance
of the same rule.

**Four conventions bind every row, and they exist because an audit trail that carries personal data is
a second copy of the thing it audits.**

- **The actor is the ASP.NET Core Identity `UserId` — the opaque GUID — never the login name.** That is
  06 §9.2's decision and this document's own finding is why: §3.11 records that `Order.Username`
  [src/MVC5/MvcMusicStore/Models/Order.cs:18] stores the login name and is the link between an order's
  nine personal-data fields and a person. A login name in a log record therefore reproduces exactly the
  identifier that makes the personal data attributable, in a store with different access controls and a
  different retention period. The `UserId` is pseudonymous: resolvable to a person only by someone who
  can already query the Identity store.
- **Where no identity exists, the actor is a bounded literal, never a submitted value — and there are
  exactly two literals, because [06 §9.6.3](06-azure-hosting-recommendations.md) — the register that governs the actor domain — declares exactly two.**
  `UserId` where an identity resolved, and **`anonymous`** where the request carries none. That covers an
  unauthenticated request reaching a handler **and** a sign-in attempt naming an account that does not
  exist: both are requests with no authenticated identity, so both write `anonymous`. *An earlier form of
  this bullet added a third literal, `unresolved`, for the second case. It is withdrawn rather than
  softened. Nothing was gained by it — the distinction it drew is already carried, twice over, by the
  `AccountNotFound` outcome value of class 2 below and by `SubjectPseudonym`, which correlates the attempts
  — and a third literal is a divergence from the owning deliverable's closed set, which is exactly the kind
  of drift a consumer document introduces by restating a decision instead of citing it.* A submitted
  identifier is attacker-controlled input and is not written to the log in any form. Where correlation
  across failed attempts is needed — the credential-stuffing case — the event carries `SubjectPseudonym`, a
  keyed HMAC-SHA256 of the submitted identifier under the exact canonical input
  [06 §8.4.7](06-azure-hosting-recommendations.md) fixes — "normalized" is not left as a word here, because
  two records for one identifier are equal only if every implementation normalizes identically, and that
  procedure, its step order and its domain separation are that sub-section's. It is stable across attempts
  within one key epoch, comparable between records, and not reversible by a reader of the log. `HMACSHA256` is in the
  shared framework, so this introduces no package — but the **key is a real cryptographic dependency and is
  specified as one** immediately below, because a keyed construction whose key is left implied is a design
  with a hole in it.

  One clarification, because the rejection contract below uses two further literals and they are not a
  third and fourth exception to this rule. 06 §9.6.3's actor register governs **application** log records; the
  provisioning record `PROV-6001` is the scoped exception 06 §9.6.3 itself grants — written by a tool rather
  than by the web application, to the separate destination 06 §9.5.1 assigns it — so its `unattributed` and
  `rejected` sentinels sit outside the two-literal set rather than extending it. No application-produced
  class in this catalog uses them.
- **Every row's field list is exhaustive and closed.** A field not listed is not permitted, which is what
  makes the redaction tests of 06 §9.2 able to fail. **The `Severity` column and 06 §9.6.1's category pinning
  are two different mechanisms and they do not conflict**, which is worth one sentence because they use the
  same words: `Severity` here is the level at which each individual record is written, and 06 §9.6.2's
  control 1 pins the reserved security-event *category* at `Information`, which is a **floor** on what that
  category emits. Every level this catalog uses — `Information`, `Warning`, `Error` — is at or above that
  floor, so no class below is filtered out by it, and no class is written below `Information` precisely so
  that none ever could be. `Timestamp` (UTC, per deliverable 05 §8.13),
  `EventId`, `Outcome`, `Severity` and a **correlation identifier** are implicit on every row and are
  not repeated in the field column. **That correlation identifier has two domains rather than one**, because
  three of the sixteen classes are not produced by a web request at all — **two of them by tooling, and one
  by nothing this plan builds** (§6.8.1.1): for the **thirteen**
  application-produced classes it is the request's **W3C trace identifier** — the 32-hexadecimal-character
  `trace-id`, which is what [06 §9.5](06-azure-hosting-recommendations.md)'s `TraceId` column is sized for,
  and never the whole `traceparent` header; for the **two** tool-produced classes there is no request, and
  it is the **run identifier of the invocation** — the provisioning run for `PROV-6001`, and
  the provisioning run or the Identity-migration run for `AUTHZ-3001` — rendered in the same
  32-hexadecimal-character form. The reserved sixteenth class, `AUTHZ-3002`, has **no** correlation domain
  assigned here, because a domain is a property of a producer and it has none; whichever producer a future
  revoke capability puts behind it fixes its domain then, and until then no row of either column can carry
  it. [06 §9.5](06-azure-hosting-recommendations.md) carries those two domains as
  **two** nullable columns rather than one shared column — `TraceId` for the HTTP domain, `RunId` for the job
  domain, with a check constraint requiring exactly one of them populated per row and selecting which by the
  `Producer` column — so which domain a stored value belongs to is structural rather than inferred from its
  shape. *An earlier form of this bullet said "a request correlation identifier" without qualification. That
  is unsatisfiable for a record a command-line tool writes, and it would have left the correlation of the
  non-web classes to whatever the implementer had to hand.*
- **Every externally supplied value that reaches a record is validated by its own domain, and no record is
  a line of prose.** **Three** fields in this catalog carry a value that arrives from outside the emitting
  program, and they arrive on **two** channels, both of which
  [05 §10.2](05-aspnet-core-migration-approach.md) closes: the **actor** on
  **`MUSICSTORE_AUDIT_ACTOR`**, an **invocation-scoped environment variable** which that section states is
  **not a switch** — the `admin` verb's accepted switch set is closed at four and none of them is an
  actor, so an invocation that passed the actor on the command line is rejected by that section's closed
  pre-parse before any host is built — and the **target username** and the **role name** as the
  `--username` and `--role` switches of that same closed set
  ([05 §10.2](05-aspnet-core-migration-approach.md) properties 1b, 3 and 4 — the closed switch set, the
  operations that consume the two named values, and the record they land in). An earlier form
  of this convention called the actor the only such field, which was wrong and
  left two channels unguarded; *an earlier form also called all three of them **arguments** of the
  command, which was wrong in the other direction — it required of the actor exactly the invocation shape
  that section's pre-parse rejects.* *An earlier form also cited a "property 3a" alongside those two. [05
  §10.2](05-aspnet-core-migration-approach.md) defines properties 1, 1a, 1b, 2, 2a, 2b, 3, 4 and 5 and no
  property 3a, so that citation named nothing; it is withdrawn here and wherever else this section carried
  it, and §6.8.1.1's note below the producer table states what the withdrawal costs.* An audit field taken from either channel and interpolated into a log line is
  the textbook log-injection defect, **CWE-117**: a value carrying a line break forges a second record, and
  one carrying a field separator or a plausible field name rewrites the record it sits in — which in an
  audit trail means an operator can fabricate the evidence of their own action. **Five rules close it, and
  a value is validated against the domain of the field it lands in rather than against one common
  filter** — a username and a role name are not actor identifiers and a single permissive character class
  for all three would be the weakest of the three checks applied to all of them:
  - **Actor authority is a precedence, not a preference — and the supplied claim is recorded separately,
    always.** Where the execution platform exposes a run-initiator identity the command can read for
    itself — a release pipeline's own run-initiator variable — **that value is the record's `Actor`**, and
    the supplied `MUSICSTORE_AUDIT_ACTOR` value **never** becomes the `Actor` on that run. Where the
    platform exposes no such
    metadata, which is the manual run 06 §9.5 admits only under its break-glass conditions, the validated
    variable value is the `Actor`. In **both** cases the supplied value is also written to a
    **separate `SuppliedActorClaim` field**, so a supplied value can never overwrite an attested one and a
    reviewer sees both — including the case that matters, an operator whose claim **disagrees** with the
    platform's attestation, which is visible in the record rather than lost in a precedence rule. Because
    the two fields are equal both when an attestation agrees with the claim and when there was no
    attestation at all, every record additionally carries **`ActorSource`**, one of exactly two literals —
    `platformMetadata` or `environmentVariable` — naming which channel produced the `Actor`. Without it the
    precedence is unreadable after the fact, and "the actor is attested" becomes an assumption rather than
    a recorded property.
  - **The channel is a variable rather than a switch, and that is a property of this control rather than
    an implementation detail of the tool.** The reason
    [05 §10.2](05-aspnet-core-migration-approach.md) puts the actor on an invocation-scoped environment
    variable is the reason this document cares about most: a value on a command line is **visible in
    process listings to any user on the host and is recorded by shell history and by pipeline logs**,
    which is precisely why that section refuses to accept the password there either. An
    invocation-scoped variable is in **none** of those three places, so the channel that carries the
    attribution of an administrative privilege change is not itself a disclosure surface. The property
    this document requires of the channel is unchanged by the move and is strengthened by it: the run is
    **refused without an actor** — an absent, empty or whitespace-only value fails the invocation before
    any work, per that section — so no provisioning record is ever unattributed, and the value's
    validation is the closed character set and hard bound below rather than the channel's own shape.
  - **The actor and the supplied claim: a closed character set and a hard bound.** Letters, digits and
    exactly five further characters — `-`, `.`, `_`, `@` and `\` — which together cover a UPN, an
    email-shaped identifier and a `DOMAIN\account`; then a hard bound of **128 characters**, applied after
    the character-set check. The same two checks apply to `SuppliedActorClaim`, so recording the claim
    separately does not create an unvalidated channel of its own.
  - **The target username: the application's own allowed character set, checked before any store call —
    and this document states no second policy.** The set is **not** the target framework's default.
    [05 §6.1](05-aspnet-core-migration-approach.md) owns the single user-name policy for this application,
    and [05 §10.2](05-aspnet-core-migration-approach.md) requires this command to validate `--username`
    against **that** set rather than against the framework default — `--username` being the spelling that
    section's closed switch set fixes, which this document cites rather than respelling. An earlier form of
    this bullet wrote the switch as `--user-name`, a spelling the closed pre-parse of that section does not
    accept. An earlier form also restated
    the framework default here, which is **wider** than the owned policy, so a name the application would
    refuse at registration would have passed the command's check — two policies for one field, which is how
    the two drift. This document therefore cites the owner and adds nothing to it. A value outside the
    owned set is rejected, which is both a log-injection defence and a correctness one: a name the
    application's own options would refuse can never name an account in the store, so refusing it before
    the store call turns a confusing downstream failure into a stated rejection. This field is the second CWE-117 channel precisely because
    row 16 is the one class permitted to carry a login name, and because it is written on every outcome the
    run **reaches** — including `Failed_UserNotFound`, where no account exists and the value in the record
    is nothing but the operator's own input, having passed validation. **"Every outcome the run reaches" is
    not "every outcome", and the difference is the whole rejection case.** A value that fails this check is
    never written to this field in any form, so the outcome that records such a failure —
    `Failed_ArgumentRejected` — carries a bounded sentinel here instead. Requiring a *validated* value on an
    outcome that exists because the value was *not* valid is not a strict rule, it is an unsatisfiable one,
    and an implementer facing it writes the submitted value. The rejection contract below the catalog table
    states the sentinel and the codes that replace it.
  - **The role name: the closed role set, not a character class.** The target defines exactly **one**
    role, `Administrator` — the only role any edition's authorization guard names (§3.2, §4.2, §5.2), as
    at [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:12], and the only role either
    provisioning path creates
    [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:31-36],
    [src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:32-33]. A role name is therefore checked for membership
    of that closed set rather than for permitted characters, and anything else is rejected before any store
    call. A closed set is the stronger check wherever the domain is finite and known, and it is what keeps
    a mistyped role from silently creating a second administrative role nothing guards.
  - **Rejection is a non-zero exit that changes nothing, and a rejected value is never repaired.** A
    **control character or a line break is rejected outright rather than stripped or escaped**, in every
    one of the three fields: a value containing one is not an identifier, and quietly repairing it destroys
    the only evidence that something tried. An over-long value is **rejected rather than truncated** — a
    truncated value is a different value, and silently becoming a different actor, account or role is the
    same defect in a quieter form. Every rejection exits non-zero having changed nothing, accompanied by
    **exactly one** record naming the failure category and the field that failed, and **not** echoing the
    offending value, so a rejected invocation is itself audited without the rejected value reaching the
    sink. Three things make that satisfiable rather than merely stated — a bounded sentinel wherever a
    field's own value is what failed, two closed code sets in place of the value, and a fixed validation
    order so that *which* field is reported is not left to the implementation — and all three are fixed by
    the **rejection contract** below the catalog table.

  And the emission form, which is the defence that does not depend on the validation being perfect:
  **every record in this catalog is emitted as a structured object — one JSON object per record, each
  field a named property — never as an interpolated message string.** A value that somehow passed
  validation still cannot forge a record boundary or truncate a line inside a JSON string, because the
  serializer escapes it. It also fixes the test contract: the assertions of §6.8.1.1's acceptance criteria
  **parse each record and assert on its fields structurally**, and never match text in a rendered line,
  so no test can pass on a substring that happens to appear in the wrong field.

**The `SubjectPseudonym` key — the whole lifecycle, because the construction is worthless without it.**
`SubjectPseudonym` is the only value in this catalog computed with a key, and a keyed value whose key has
no name, no origin, no reader and no rotation policy is not a control — it is an assumption that somebody
downstream will invent one. The key is therefore **required configuration of the ported application**, on
the same footing as the connection string.

**Ownership, stated before the table because it decides what this table may say.**
[06 §8.4.7](06-azure-hosting-recommendations.md) owns **the key record, the label grammar and the key
lifecycle** — the structured versioned key-set document the secret holds, the label grammar and
its digest input, the canonical input procedure, additive rotation and non-destructive rollback, the
ordered fail-closed startup validation, and the defined read-back outcomes with their tests. It says *what
valid means*. This document owns the application-side contract: **which event classes carry a
`SubjectPseudonym`, over what subject, why the value is keyed at all, and that the application fails closed
on an invalid key** — and it cites 06 §8.4.7 for every mechanism element rather than restating one. The
table below is that contract; where a row touches mechanism it points at 06 §8.4.7 and stops there.

*An earlier form of this section specified a mechanism of its own — a single raw 32-byte setting, an epoch
identifier derived from it by a domain-separated HMAC, and a two-field `<keyId>.<payload>` label. It is
withdrawn in full rather than reconciled row by row. It was written to fix a real defect, the absence of any
binding between a key and the labels it produced, and 06 §8.4.7 now fixes that defect at the owning
deliverable with a stronger construction — the key id and the scheme version are **inside the digest
input**, and the active-key pointer and the material live in **one document**, so activation is atomic
without deriving anything. Two mechanisms for one value is the failure this document is elsewhere strict
about; the owner's is the one that stands.*

| Property | Requirement |
| --- | --- |
| **Configuration key** | `Security:EventLog:SubjectPseudonymKey`, read through the configuration abstraction like any other setting — never a compiled constant, never a file in the payload. The **value** it resolves to is 06 §8.4.7's key record, not bare key material; that setting name and its platform binding are 06 §8.4's |
| **Value and generation** | **The structured versioned key record of [06 §8.4.7](06-azure-hosting-recommendations.md)**, whose contents, field names and shape are that sub-section's and are **not** reproduced here. Two requirements are this document's and are stated rather than cited, because 06 §8.4.7 attributes them to it and applies them per key: each key is **exactly 32 bytes (256 bits)**, and each is produced by a **cryptographic random generator**. Created per environment by the deployment principal at provisioning time. What this document requires of it, and 06 §8.4.7 satisfies: it is **not** derived from another secret, **not** reused across environments, and **not** taken from the data-protection key ring — that ring rotates on its own schedule (06 §7.4), which would silently change pseudonym values and break comparability without anyone deciding to |
| **Storage and delivery** | A secret in the platform secret store, surfaced to the application as a configuration reference resolved by the site's managed identity. It never appears in source, in `appsettings.json`, in the deployment payload or in a pipeline log. Mechanism and grants: **06 §8.4** |
| **Who may read it** | The application's runtime managed identity, for `get` on that one secret, and the deployment principal for the create-and-rotate operation. No standing human access. The value is never read back into a pipeline variable, because a pipeline variable is a log entry waiting to happen |
| **Scope** | **One key record per environment, and per deployment slot within an environment**, marked as a setting that does not move in a slot swap (06 §8.4, which applies the same slot-scoping rule as 06 §7.5 control 3). A shared key would let a lower-trust environment compute production pseudonyms and so confirm, by comparison against a production log, whether a chosen identifier appears in it — which is exactly the reversal the pseudonym exists to prevent. **The enforcing control is data, not diligence**: the record carries an `environment` tag and the application refuses to start where it does not match its own environment (06 §8.4.7, startup check 3), so a misconfigured reference is rejected before a single label is minted rather than discovered by comparing logs afterwards |
| **Rotation** | On a stated interval set by the security owner, and **immediately** on suspected exposure. Rotation is not free and its cost is stated rather than discovered: pseudonyms computed under a new key do not equal those computed under the old one, so **correlation does not survive rotation**. The operation itself is 06 §8.4.7's — additive, one secret version, the previous key retained and the active pointer moved in the same write, so there is no interval in which the active key id names material the application cannot resolve, and existing labels stay interpretable |
| **Correlation across rotation** | Handled by labelling the value rather than by adding a field: the emitted value is **06 §8.4.7's label, whose grammar that sub-section fixes and this document does not reproduce** — what matters on this side is the property it guarantees, that the scheme version, the environment tag and the key id are **part of the digest input** rather than a prefix, so an altered label fails verification instead of verifying under a different rule set. A reader can therefore tell that two records belong to different epochs instead of concluding they are different subjects, and **correlation is defined as within-epoch only**. The rotation interval must therefore be longer than the longest analysis window this correlation serves — the security owner sets both, and sets them together |
| **Absent, malformed or short key — fail closed** | The application **fails to start**, and reports unhealthy if it starts at all, exactly as 06 §8.4 requires of any unresolvable configuration reference. **What "malformed" means is 06 §8.4.7's ordered startup validation**, whose checks and their order that sub-section enumerates and this document does not restate; what this document requires is **the whole of that sequence rather than a length check**, and specifically that it runs before any label is minted. It must **never** degrade: not to logging the submitted identifier, not to an unkeyed digest — an unkeyed SHA-256 of a login name or email is reversible by enumeration and is pseudonymisation in name only — and not to omitting `AUTH-1002`, which is a required class of this catalog. A deployment gate proves the record resolves and validates **before traffic is admitted** (06 §8.4, using the deployment gate of 06 §9.3) |
| **Never logged** | The key itself is in 06 §9.2's never-logged secret class, which already names it. That rule is 06's and is not restated here |

**What this document adds to 06 §8.4.7's mechanism — four requirements the mechanism cannot infer, because
they are properties of the events rather than of the key.**

**1. Two classes carry the field, and both carry it over the same subject type.** `AUTH-1002` carries
`SubjectPseudonym` on every outcome, and `ACCT-2001` carries it on its two failure outcomes; no other class
in this catalog carries it. In both, the pseudonymized value is the **identifier as submitted, before any
store lookup**, because the field exists to correlate attempts naming accounts that may not exist — which
is the whole of its value in the credential-stuffing case, and is why the field survives the actor being
`anonymous`. In 06 §8.4.7's canonical input that fixes the **subject type** as `submitted-name` for both
classes, and this document requires that one value rather than leaving the choice to the emitter: two
classes computing one identifier under two subject types would produce two unequal labels for one subject,
which is precisely the split 06 §8.4.7's domain separation exists to make impossible between *different*
subjects and must not introduce between the same one.

**2. An empty canonical result omits the field; it never produces a label.** 06 §8.4.7's canonical input
rejects an empty result, and this is what "rejects" means at the emission site: the record is still written
— `AUTH-1002` is a required class and a blank submission is exactly the attempt worth recording — and it is
written **without** `SubjectPseudonym`. Omission is required rather than optional because the alternative is
one constant label shared by every empty submission, which is a correlation that is false rather than
absent, and a false correlation in this field merges subjects silently.

**3. Correlation is within-epoch only, and the analysis path must say so rather than infer a subject.** The
label's own scheme version, environment tag and key id make an epoch boundary visible, and 06 §8.4.7 fixes
the three outcomes of reading a label back — match, no match, and **uninterpretable**. The requirement this
document places on the security-analysis path is the third one: a label that does not parse, whose scheme
version is unknown, whose environment tag is not this environment's, or whose key id does not resolve is
reported **uninterpretable and distinctly from "no match"**, and is never counted as a subject. Collapsing
the two is how an audit query merges or splits subjects without saying it has; and because this document
reads per-subject counts off this field — `ConsecutiveFailureCount` on `AUTH-1002`, and the unthrottled
guessing that F-09-03 and §3.3 establish is detectable only by grouping attempts by subject — a silent
merge or split is a security conclusion drawn from a parse error.

**4. The field is never the whole of a control.** `SubjectPseudonym` makes repeated attempts *comparable*;
it does not make them *rate-limited*, and no finding in this document is closed by the field's existence.
That is stated because a keyed correlation identifier reads like a mitigation and is not one — the
throttling requirement stands on its own, under the finding that names it.

**Where 06 §8.4.7 and this section could drift, and the one rule that prevents it.** Every mechanism value
above is cited, not restated: the record's fields, the label's grammar and character counts, the digest
input and its domain separation, the canonical procedure and its step order, the rotation and rollback
operations, and the six startup checks are 06 §8.4.7's, and a figure or literal from any of them appears in
this document only as a citation of that sub-section. Where the two disagree, **06 §8.4.7 governs and this
document is wrong** — the same rule §6.8.1's ownership paragraph applies to the rest of the observability
contract.

**Why a keyed pseudonym at all, since the alternatives are cheaper.** Three were considered and each fails
on a stated ground rather than on taste. Emitting **nothing** for an unresolved sign-in loses the only
signal that distinguishes one person mistyping a password from an enumeration run across many accounts,
and the client address does not substitute for it, because a distributed attempt varies the address and a
shared egress makes many users look like one. An **unkeyed hash** is reversible by enumeration: login
names and email addresses come from a small, guessable space, so anyone holding the log can confirm
whether a chosen identifier appears in it. **Deriving the key from an existing secret** — the connection
string, or the data-protection ring — reuses key material across purposes and, in the ring's case, ties
pseudonym stability to a rotation schedule set for an unrelated reason. A declared, required, rotatable
key is the cheapest option that actually holds, and the cost of declaring it is the table above.

**The durable representation every row below requires — because a closed field set the store cannot hold is
not closed.** [06 §9.5](06-azure-hosting-recommendations.md) owns `dbo.SecurityAuditLog`: its columns, their
types and widths, its constraints and its grants. This document declares no table and no column type. What
it must state, because it is the *producer* whose records that table exists to hold, is the representation
each record needs in order to survive the insert at all — and it is stated here, once, rather than per row.
Four requirements, each of which the owner's DDL as it now stands satisfies; a producer requirement that the
store silently truncates or rejects is an audit gap that appears at run time and not at review time, which is
why each one is stated here and checked against that DDL rather than assumed to be met.

| # | What this catalog produces | The representation it requires of 06 §9.5's table |
| --- | --- | --- |
| 1 | **Exactly three severity levels across the sixteen classes: `Information`, `Warning` and `Error`** — the closed set the third convention above states, and no class is written below `Information` | The `Severity` domain must **admit all three**, and [06 §9.5](06-azure-hosting-recommendations.md)'s `CK_SecurityAuditLog_Severity` on `CREATE TABLE dbo.SecurityAuditLog` **admits them** — `Error` included, which an earlier form of that constraint omitted. `Error` is used by exactly **two** classes, and both are writes that did not complete: `ORDER-5002`, a confirmed-order write that failed, and `ADMIN-4003` **at `AbsentRow` only** — row 13 is the one row whose severity is per outcome, its `Success` staying at `Warning`. A domain that omitted `Error` would not downgrade either record, it would **reject the insert**, so the two classes whose failures most need a durable trail would be the two that have none. The requirement is that the three levels this catalog emits are all accepted; whether the domain also admits levels this catalog never writes is the owner's to decide |
| 2 | **An `Actor` and a `SuppliedActorClaim` validated to a closed character set and a hard bound of 128 characters**, with a longer value **rejected rather than truncated** (fourth convention, and row 16) | The `Actor` column must hold **128 characters**, and [06 §9.5](06-azure-hosting-recommendations.md)'s is declared `nvarchar(128)`, which **holds them**. Truncation is not an acceptable degradation of an audit actor: a truncated `DOMAIN\account` or UPN names a different principal, or no principal, and the record then attributes an administrative action to something that does not exist — which is precisely the failure the 128-character rejection rule upstream exists to prevent. A store bound narrower than the validated bound would move the rejection from the command, where it is reported to an operator, to the insert, where it is a lost record |
| 3 | **A correlation identifier in two domains** — the request's W3C `trace-id` for the thirteen application-produced classes, the run identifier for the two tool-produced ones (third convention above; the reserved sixteenth class populates neither, because nothing emits it) | **Two columns, each 32 characters and each nullable**, with exactly one populated per row. [06 §9.5](06-azure-hosting-recommendations.md) declares them and this document requires that form: `TraceId` for the HTTP domain, `RunId` for the job domain, each constrained to exactly 32 lowercase hexadecimal characters, with `CK_SecurityAuditLog_Correlation` enforcing the exactly-one rule and selecting which column by the `Producer` value. The requirement this catalog places on the store is that the correlation not be **request-only**, because three of its sixteen classes have no request — and the owner's two columns express that better than one shared column could, since a single column carries a value with no statement of which domain produced it. **The split is `13 + 2 + 1`, and it is written that way here for the same reason §6.8.1.1 gives**: thirteen application-produced classes populate `TraceId`, the **two** tool-produced ones populate `RunId`, and the reserved sixteenth populates **neither**, because nothing emits it — so the three request-less classes are not three producers, and a row that said "three tool-produced" would put a column requirement on a class with no producer to place it. An earlier form of this row asked for one column serving both domains; that request is withdrawn in favour of the owner's pair |
| 4 | **Every permitted field beyond the implicit set, as structured properties** — never interpolated prose (fourth convention) | The columns carry the implicit set and the actor, target and outcome; **every remaining permitted field of a class lands in the JSON field column**. For the widest row, class 16 on `Failed_ArgumentRejected`, that is `RoleName`, `Operation`, `SuppliedActorClaim`, `ActorSource`, `FailureCategory`, `RejectedField` and `RejectionRule` — seven bounded values, each a code or a bounded identifier and none a free-form message, which is what keeps the widest record inside a kilobyte-scale bound. A class whose fields would not fit is a design error in this catalog, not a reason for the store to spill to prose |

**Where this catalog and the owner's schema meet, and why the requirement is stated anyway.** Under the
ownership rule of §6.8.1 the owner governs a *decision*; a **producer requirement** is not a decision the
owner can take on the producer's behalf, which is why requirements 1 and 2 above are stated here at all
rather than left to whatever the schema happens to declare. Both are **satisfied by the DDL
[06 §9.5](06-azure-hosting-recommendations.md) now publishes**: in its `CREATE TABLE dbo.SecurityAuditLog`
the `Actor` column is `nvarchar(128)`, and `CK_SecurityAuditLog_Severity` admits `Error` alongside the other
two levels this catalog emits. An earlier form of that DDL satisfied neither, and both were recorded here as
corrections required before the table is created at [06 §6.3](06-azure-hosting-recommendations.md) step 1;
the corrections have been made, and these two rows are now checks against the owner's current text rather
than outstanding work against it. What does not change is that neither would ever have been worked around on
this side — writing `Critical` where this catalog says `Error`, or truncating an actor to fit, would each
replace a schema defect with a false record, which is the reasoning that makes the requirement legible
whether or not the schema currently meets it.

| # | Event class | Event id | Actor | Target | Outcome values | Severity | Permitted fields beyond the implicit set |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | Authentication succeeded | `AUTH-1001` | `UserId` | the same `UserId` | `Success` | Information | `PersistentCookieRequested` (bool); `ClientIpAddress` |
| 2 | Authentication failed | `AUTH-1002` | `UserId` where the account resolves, else `anonymous` — the second of [06 §9.6.3](06-azure-hosting-recommendations.md)'s two literals, with the account's non-existence carried by the `AccountNotFound` outcome and the correlation by `SubjectPseudonym`, not by a third actor literal | as actor | `InvalidCredentials`, `AccountLockedOut`, `AccountNotFound` | **Warning** | `SubjectPseudonym`; `ClientIpAddress`; `ConsecutiveFailureCount` |
| 3 | Account locked out | `AUTH-1003` | `UserId` | the same `UserId` | `LockedOut` | **Warning** | `FailedAttemptCount`; `LockoutEndUtc` — the lockout expiry, written with the round-trip (`o`) specifier; `ClientIpAddress`. **Delivery is at-least-once, not exactly-once**, and every physical record carries the durable de-duplication key `{UserId}\|{LockoutEndUtc:o}`: two physical records with equal keys are **one logical event**, and a reader collapses on exact key equality. The mechanism is seam 3 of §6.8.1.1, which states this contract in the same words |
| 4 | Signed out | `AUTH-1004` | `UserId` | the same `UserId` | `Success` | Information | `ClientIpAddress` |
| 5 | Account registered | `ACCT-2001` | `UserId` of the new account on `Success`; **`anonymous` on either failure outcome**, because no account was created and the request is unauthenticated | the same `UserId` on `Success`; **`none`** on either failure outcome | `Success`, `DuplicateUserName`, `ValidationFailed` | Information | `ValidationFailureCategory` — a code, never the submitted values; `SubjectPseudonym` on the failure outcomes, so repeated attempts against one name are correlatable without writing it; `ClientIpAddress` |
| 6 | Password changed | `ACCT-2002` | `UserId` | the same `UserId` | `Success` | **Warning** | `ClientIpAddress`; `RehashedOnSignIn` (bool) where the write was a migration rehash on sign-in rather than a user action. **This field is set only from the rehash seam §6.8.1.1 specifies, and only on a store-observed successful hash change** — never from the hasher's `SuccessRehashNeeded` advice, which is produced before persistence is attempted, and never inferred from a successful sign-in. A rehash whose store update failed produces **no** record of this class, only the distinct diagnostic that seam names. If the seam is not built the field is not permitted, because a guessed value in an audit record is worse than an absent one |
| 7 | Password change failed | `ACCT-2003` | `UserId` | the same `UserId` | `IncorrectCurrentPassword`, `PolicyRejected` | **Warning** | `PolicyRuleViolated` — the rule identifier, never the candidate password; `ClientIpAddress` |
| 8 | Role granted | `AUTHZ-3001` | The **validated actor** of §6.8.1's fourth convention, by the same precedence as row 16: the run-initiator identity the platform attests where it attests one — which is the Identity data migration's only source, since it reads no actor variable — otherwise the provisioning command's validated `MUSICSTORE_AUDIT_ACTOR` value. **Not** a `UserId` taken from a web request: §6.8.1.1 assigns this class no application producer, because nothing in the ported application grants a role. `deploymentPrincipal` is one value a pipeline may supply, not a fixed literal | `UserId` of the grantee | `Success`, `AlreadyHeld` — the command emits `Success` only, on the run that actually adds the membership; `AlreadyHeld` is the migration's outcome for an assignment already present in the target (§6.8.1.1) | **Warning** | `RoleName` — a member of the closed role set; `ActorSource`; `SuppliedActorClaim` where the producer reads the actor variable, which the migration does not, so the field is **absent** on its records rather than empty |
| 9 | Role revoked | `AUTHZ-3002` | **Reserved: this class has no producer, so it has no actor.** Nothing in the target withdraws a role. [05 §10.2](05-aspnet-core-migration-approach.md) property 3 states the provisioning command never deletes or demotes anything; its verb table is closed and contains no revoke verb; and the `admin` verb's accepted switch set is closed at four, none of which asks for a revocation. An earlier form of this row attributed the class to a "revoke mode" of the provisioning command, specified in a "property 3a" of [05 §10.2](05-aspnet-core-migration-approach.md) — a property that section does not define and a mode it does not describe — so the attribution is **withdrawn** rather than re-pointed at some other producer, because there is none to re-point it at. §6.8.1.1's note below the producer table states what that costs and who owns closing it | reserved with the class | reserved with the class. `Success` — an **actual** removal — is the only outcome it would ever carry, and it is **unreachable today** because no operation performs a removal. No other class silently absorbs the fact: there is no attempt to record either, since nothing attempts it | **Warning** | reserved with the class: `RoleName` — a member of the closed role set; `SuppliedActorClaim`; `ActorSource`. The field list is stated rather than left blank so that a future producer inherits a defined record shape instead of inventing one, and so [06 §9.5](06-azure-hosting-recommendations.md)'s table needs no change when it arrives |
| 10 | Authorization denied | `AUTHZ-3003` | `UserId`, or `anonymous` | the route reached — controller and action names | `Denied` | **Warning** | `DenialKind` — `Challenged` or `Forbidden`, the two closed values §6.8.1.1's seam 2 reads from `PolicyAuthorizationResult`, separating an unauthenticated caller from an authenticated one who is not permitted; `RequiredPolicy` — **derived from the endpoint's `IAuthorizeData` metadata and validated against a closed set**, by the four-step rule in seam 2, and **not** from the `AuthorizationPolicy` argument, which carries no name; `ControllerName`; `ActionName`; `ClientIpAddress` |
| 11 | Album created | `ADMIN-4001` | `UserId` | `AlbumId` on `Success`; **`none`** on `ValidationFailed`, because no row was inserted and no key was allocated | `Success`, `ValidationFailed` | Information | `AlbumId` on `Success` only; `GenreId`; `ArtistId`; `ValidationFailureCategory` on `ValidationFailed` — a code, never the submitted field values |
| 12 | Album updated | `ADMIN-4002` | `UserId` | `AlbumId` | `Success`, `NotFound`, `ValidationFailed` | Information | `AlbumId`; the **names** of the properties changed — never their before and after values, which may include price |
| 13 | Album deleted | `ADMIN-4003` | `UserId` | `AlbumId` | `Success`, `AbsentRow`. **`NotFound` is a retired spelling for this record id**, named here so that a citation of it resolves instead of reading as a live third value; it stays live elsewhere in this catalog, including row 12 above, where it names a genuine 404. The retirement is scoped to this id and not to the word: `DeleteConfirmed`'s null branch answers a **preserved 500**, not a 404 ([05 §4.15](05-aspnet-core-migration-approach.md) census site F5, and F-09-37 in §6.3), so an outcome word borrowed from a status the site does not return would make the record contradict the response beside it | **Per outcome — `Success` at `Warning`, `AbsentRow` at `Error`** — and this is the one row in the catalog whose severity is not single-valued. `Warning` for a completed deletion, because it is the administration operation with no undo. `Error` for the absent-row branch, because that record is the only one accompanying a served **500**, and writing it at `Warning` would leave a served 500 invisible to every error-severity alert. **The split of ownership is settled, not held open:** the *behaviour* — the preserved 500, the single record and this severity pair — is [05 §6.5](05-aspnet-core-migration-approach.md)'s, and the *vocabulary* is this catalog's, which is exactly the division that section states | `AlbumId` |
| 14 | Order placed | `ORDER-5001` | `UserId` | `OrderId` | `Success` | Information | `OrderId`; `LineItemCount`; `OrderTotal`; `PromoCodeApplied` (bool — **not** the code). The boolean is [06 §9.2](06-azure-hosting-recommendations.md)'s sanction rather than this document's addition: that section's never-logged table places the promo code under business inputs and states that its *application* is recorded as a boolean and its value never, and its closed order-record list names the boolean explicitly as part of that list rather than as an addition to it. The two statements in 06 agree, and this field is written under both |
| 15 | Order placement failed | `ORDER-5002` | `UserId` | `OrderId` where one was allocated, else `none` | `ValidationFailed`, `EmptyCart`, `PersistenceFailed` | **Error** | `FailureCategory`; `LineItemCount`. **The exception detail goes to the error log, not to this record**, and never onto a response-bound channel — F-09-14 and §4.9 |
| 16 | Administrator provisioning operation | `PROV-6001` | the **validated actor** of §6.8.1's fourth convention, by its **precedence**: the run-initiator identity the execution platform supplies **is** the `Actor` wherever the platform supplies one; only where it supplies none is the `Actor` the validated value the command requires on **`MUSICSTORE_AUDIT_ACTOR`**, the invocation-scoped environment variable [05 §10.2](05-aspnet-core-migration-approach.md) property 4 makes the actor's only channel — that section states in as many words that it is **not** a switch, and an invocation that passed the actor on the command line is rejected by its closed pre-parse before any host is built. The supplied value is **always** recorded separately as `SuppliedActorClaim`, and `ActorSource` names which channel produced the `Actor`, so a supplied value can never overwrite an attested one. **Both actor-side values** — the `Actor` and the `SuppliedActorClaim` — are checked against the closed character set, **rejected outright** on any control character or line break, and bounded at 128 characters with a longer value rejected rather than truncated; the target username and the role name are validated against **their own** domains, per the same convention. Every value is written as a property of a structured record rather than interpolated into a line — an unvalidated externally supplied value in an audit field is CWE-117. `deploymentPrincipal` is one such value a pipeline may supply, not a fixed literal | the **target username**, validated against the user-name set [05 §6.1](05-aspnet-core-migration-approach.md) owns — this document states no set of its own — per §6.8.1's fourth convention, and present on every outcome the run **reached**. That includes `Failed_UserNotFound`, where the name passed validation and simply named no account — an outcome **reserved with row 9's class** and unreachable on any run this plan produces, because the provisioning run creates the account it does not find ([05 §10.2](05-aspnet-core-migration-approach.md) property 3) and no other run exists; the field rule is stated so that a producer which one day resolves an account without creating it inherits it. On the one outcome where a supplied value is what failed validation, `Failed_ArgumentRejected`, the field is **declared absent** — never the submitted value, and never a reserved literal either, because every legal user name is a possible account name and reserving one would make that account unmanageable through this command: the rejection contract below the table fixes this per field, and it is the whole reason that outcome exists separately from `Failed_PolicyRejected` | `RoleCreated`, `RoleAlreadyExisted`, `UserCreated`, `Created`, `AlreadyPresent_NotRotated`, `Rotated`, `MembershipAdded`, `MembershipAlreadyPresent`, `MembershipRevoked`, `MembershipNotHeld`, `NotAttempted`, `RunSucceeded`, and the **six** closed failure outcomes `Failed_ArgumentRejected`, `Failed_UserNotFound`, `Failed_PolicyRejected`, `Failed_PublishedCredentialRefused`, `Failed_IdentityError`, `Failed_StoreUnavailable`. **Three of those values are reserved with row 9's class and are not reachable on any run this plan produces** — `MembershipRevoked`, `MembershipNotHeld` and `Failed_UserNotFound`, each of which presupposes a removal or a resolve-without-create that no operation performs — so the set is **closed at eighteen values of which fifteen are exercisable**, and the acceptance criteria below count it that way rather than demanding a test for a branch that cannot be entered. **`NotAttempted` and `RunSucceeded` are the two values the four-record invariant requires** and neither is optional: `NotAttempted` is what an operation record carries on an invocation that never reached that operation — which is **every** rejected invocation, all three of whose operation records carry it — and `RunSucceeded` is what the fourth record, the run record, carries where every operation the run reached completed. **`UserAlreadyExisted` is a retired value rather than a nineteenth surviving one**, and it is named here so that a citation of it resolves: [05 §10.2](05-aspnet-core-migration-approach.md) property 4 gives the credential no record of its own, so the account-existence discovery and the credential verdict land on **one** record carrying **one** outcome — `UserCreated` where the account was absent, `Created` where it existed with no password at all, `AlreadyPresent_NotRotated` or `Rotated` where it existed with one — and a separate account-existence value would be a second outcome for that single record, which is why the four above partition the operation and the retired value overlapped three of them. **The audit envelope's own outcome field and its two closed sets are that property's, not this row's** — seven distinct values across the three operation records and the run record, one value being common to both sets — and this catalog restates none of them; the values here are the **security-event** outcome domain that [06 §9.5](06-azure-hosting-recommendations.md)'s column carries and that its register defers to this row to define. `Failed_ArgumentRejected` is the **input-validation** failure and `Failed_PolicyRejected` is now **only** a credential refused by Identity's password policy: an earlier form merged the two, which made the record unreadable — an auditor could not tell a weak password from an operator's typo — and, worse, made the row unimplementable, because the merged outcome required a validated actor and target on a branch that exists precisely because a value was not valid. The rejection contract below the table states the field rules for that outcome. `Created`, `AlreadyPresent_NotRotated` and `Rotated` are the **credential verdict**, carried on the `create-user` record because that is the operation [05 §10.2](05-aspnet-core-migration-approach.md) property 4 gives the credential, and read together with the Operation field: a credential is set only where the account is absent or where the release explicitly asks for a rotation ([06 §12.1](06-azure-hosting-recommendations.md)), so a steady-state run records the not-rotated outcome and changes nothing. Which outcomes each operation may record is the mapping below the table | **Warning** | `RoleName` — a member of the closed role set, per §6.8.1's fourth convention; `Operation` — one of `create-user`, `create-role`, `add-membership`, `run`, the closed discriminator [05 §10.2](05-aspnet-core-migration-approach.md) property 4 fixes and this document restates no spelling of its own for; `SuppliedActorClaim` — the `MUSICSTORE_AUDIT_ACTOR` value as supplied, validated and bounded exactly as the actor is; `ActorSource` — `platformMetadata` or `environmentVariable`; `FailureCategory` on a failure outcome, which is the outcome value itself and **not** Identity's free-form error description; and, on `Failed_ArgumentRejected` only, `RejectedField` and `RejectionRule` — two bounded codes from the closed sets the rejection contract below the table defines, carrying **which** field failed and **which** rule it broke, and nothing whatever derived from the value that failed. **This is the one sanctioned exception to the no-login-name rule**, and its cardinality is **four records per invocation, invariantly**: one for each of the three operations [05 §10.2](05-aspnet-core-migration-approach.md) property 3 checks independently — the user, the role and the membership — plus the **run record** that property 4 makes the fourth, in that order. **Four is a constant rather than a maximum**, so the count holds on the paths that fail and on the paths rejected before any host is built as well as on the successful one, which is what makes it assertable without the consumer branching on how far the run got. **That is the only run shape there is**: an earlier form of this cell counted the credential as a fourth *operation* and a "revoke run" of two operations citing a property 3a that [05 §10.2](05-aspnet-core-migration-approach.md) does not define, and neither exists — the credential has no record of its own and no revoke run can be invoked — so the count here is four on that section's decomposition and the cardinality note below the table states it for one run rather than two. The grant *fact* is `AUTHZ-3001` and is **never the same record as a `PROV-6001`** — see that note, which also carries the withdrawal of the revoke counts and the reservation of row 9's class. Its sink and retention are [06 §9.5](06-azure-hosting-recommendations.md)'s, not the application's |

> **Row 16 is an exception with a stated reason, not an inconsistency.** AAP §0.3.2 requires the
> provisioning command to leave an audit record of what it did, and the requirement that makes this row
> an exception to the no-login-name rule is that the record must name **which account** was provisioned:
> the operator running the command has to be able to confirm it, and at that moment the `UserId` may not
> yet exist. **The record's field schema — its envelope and the durable shape it lands in — is not
> restated here; it is [06 §9.5.1](06-azure-hosting-recommendations.md)'s**, together with how many
> records a run emits and where they durably land. What this row states is the requirement and the
> class's own permitted values, the set [06 §9.6.2](06-azure-hosting-recommendations.md)'s R5 and R7
> delegate here by name, and where the two differ 06 governs, per the precedence §6.8.1 accepts above.
> It is the only class in this catalog permitted to carry a login name and it is written by a tool
> rather than by the web application. Deliverable 06 §9.2 states the general rule and this exception
> together, so the two do not read as a contradiction.
>
> **And its actor is attributed to a person or a run, never to a fixed literal — under a precedence that
> keeps the two channels apart.** An earlier form of this row named `deploymentPrincipal` as the actor
> value, which conflicts with the command's own contract: [05
> §10.2](05-aspnet-core-migration-approach.md) property 4 makes the actor a **required input** the
> command refuses to run without, precisely so the record attributes the grant to a person or a pipeline
> run rather than to the identity the process happens to hold. A later form then over-corrected and
> declared the supplied value to be *exactly* the record's actor, which contradicted the same section's
> preference for platform attestation and would have let an operator's own string overwrite an attested
> identity. **Neither reading is the contract.** §6.8.1's fourth convention resolves it as a precedence
> with two fields: **platform-supplied initiator metadata is the `Actor` wherever it exists**, the
> validated supplied value is the `Actor` only where it does not, that value is **always** written
> to `SuppliedActorClaim`, and `ActorSource` records which channel the `Actor` came from. The requirement
> the actor satisfies is unchanged — the run is refused without it, so no provisioning record is ever
> unattributed — but satisfying it no longer means trusting it over an attestation, and a disagreement
> between the two is preserved in the record instead of being resolved silently. A pipeline supplying
> `deploymentPrincipal` is one instance of the metadata channel rather than the definition of the field.
>
> **A third form of this row is corrected here, and it was the one that made the requirement
> unsatisfiable.** It attributed the actor to a **command-line argument of its own** and cited
> [05 §10.2](05-aspnet-core-migration-approach.md) property 4 for it. That section requires the opposite
> and says so twice: the actor arrives on **`MUSICSTORE_AUDIT_ACTOR`**, an invocation-scoped environment
> variable rendered by the pipeline from its own authenticated trigger, and "it is **not** a switch" —
> the `admin` verb's accepted set is closed at four and an invocation that passed the actor on the command
> line is **rejected** by that section's closed pre-parse. So the earlier text demanded the one invocation
> shape the parser refuses, and it demanded it while citing the parser. The channel is also the safer of
> the two for the reason that section gives for the password: a command-line value is visible in process
> listings and recorded by shell and pipeline history, and an invocation-scoped variable is in neither —
> so nothing this row requires of the actor is weakened by the correction, and the disclosure surface the
> argument would have created is removed.

**The cardinality of a provisioning run, stated exactly, because one run emits records of two different
classes and an implementation that merges them loses the privilege-change fact.** `PROV-6001` is the
command's **own** record — one per operation the run checks, plus a **run** record for the invocation as a
whole. `AUTHZ-3001` is the
**privilege-change** fact — that a role membership was granted. They answer different
questions, they are read by different people, and 06 §9.5 may retain them differently, so **they are never
the same record**. There is **one** run shape to count, because the command has exactly one mode that
touches a membership and it only ever adds one:

| Run | Operations ([05 §10.2](05-aspnet-core-migration-approach.md) properties 3 and 4) | `PROV-6001` records | Privilege-change record | Records in total |
| --- | --- | --- | --- | --- |
| **Provisioning** | **3** independent checks — `create-user`, `create-role`, `add-membership` — each on its own record, and a fourth record for the **run** ([05 §10.2](05-aspnet-core-migration-approach.md) property 4) | **4**, invariantly and in that order: three operation records at their own outcomes plus the run record. **Four is a constant, not a maximum** — it holds where an operation fails and where the invocation is rejected before any store call, and on that rejected path the three operation records carry `NotAttempted` while the run record carries the rejection, so a reader counts records without first establishing how far the run got | **One `AUTHZ-3001`**, and only where the membership was **actually added** — the run whose `add-membership` record is at `MembershipAdded`. A run that finds the membership already present records that operation at `MembershipAlreadyPresent` and emits **no** `AUTHZ-3001`, because no grant occurred | **5** where the membership was added; **4** on every other path, including rejection |

**A second row is withdrawn from this table, and it is named rather than dropped quietly, because the
counts it carried are cited elsewhere.** An earlier form carried a **Revoke** run of two operations — a
user resolve then a membership removal — with three branches, branch cardinalities of one, two and three records, and an
`AUTHZ-3002` on the branch that removed a held membership, all attributed to a "property 3a" of
[05 §10.2](05-aspnet-core-migration-approach.md).
**No such run can be invoked.** [05 §10.2](05-aspnet-core-migration-approach.md) defines properties 1, 1a,
1b, 2, 2a, 2b, 3, 4 and 5 and no property 3a; property 3 states the command never deletes or demotes
anything; its verb table is closed and holds no revoke verb; and the `admin` verb's accepted switch set is
closed at four, none of which asks for a revocation. So the row counted records of a run that does not
exist, and **every number in it is withdrawn** — the branch cardinalities, the `Failed_UserNotFound`,
`MembershipNotHeld` and `MembershipRevoked` outcomes those branches recorded, and the `AUTHZ-3002` its
third branch produced. What survives the withdrawal is a **reservation**, not a capability: row 9 keeps
the identifier, the single outcome and the field shape so that a future revoke capability emits a defined
record instead of an invented one, and §6.8.1.1's note below the producer table carries the operational
gap this leaves and the owner of closing it.

**The surviving row's own decomposition is corrected here too, and the record count is unchanged by the
correction.** An earlier form of that row read the run as **four operations** — role, user, credential and
membership — with one record each. [05 §10.2](05-aspnet-core-migration-approach.md) property 3 checks
**three** things independently, and property 4 gives the credential **no record of its own** while adding a
**run** record as the fourth; the operation names are that section's `create-user`, `create-role` and
`add-membership`, and this document restates no spelling of its own for them. The total is four either way,
which is exactly why the earlier form went unnoticed: what it got wrong was **which** four, and therefore
where the credential verdict lands and how many records a rejected invocation emits. Row 16's cell
above states the consequence for the outcome set.

**A privilege-change record follows the actual privilege change, and that rule is what makes the
reservation safe.** A grant fact that did not happen would be a false record, so `AUTHZ-3001` follows the
actual addition — and the same rule applied to a withdrawal is exactly why the reserved class emits
nothing today rather than emitting a `Success` nobody performed. Nothing is lost to the reader by the
withdrawal, because nothing is attempted: there is no removal run whose attempt would need recording.
Row 8's `AlreadyHeld` outcome is not a
counter-example: it belongs to the **other** producer §6.8.1.1 names — the Identity data migration,
reporting an assignment it loaded that the target already held — and not to the command.

**Which outcome each operation may record, because a bare `Created` is ambiguous without it.** Every
`PROV-6001` carries `Operation`, and the closed outcome set of row 16 partitions across it. This mapping
is the enforceable form of that partition: an outcome recorded against an operation it is not listed for
is a defect. The **six** failure outcomes partition too: `Failed_IdentityError` and
`Failed_StoreUnavailable` are reachable on **any** record whose operation ran, `Failed_UserNotFound` and
`Failed_PublishedCredentialRefused` only where the rows below say, `Failed_PolicyRejected` is **only** a
credential refused by Identity's password policy and therefore lands on the `create-user` record, which is
where the credential verdict belongs, and
`Failed_ArgumentRejected` is **only** an input refused by the validation rules of §6.8.1's fourth
convention and therefore lands **only on the run record**, no operation having run. Those last two were one outcome in an earlier form of this catalog and are now separate,
because they differ in the one way that decides what the record may contain: a policy-rejected credential
arrives on a run whose actor and target both validated, while an argument rejection is by construction a
run in which at least one of those values did not.

| `Operation` | Non-failure outcomes | Reachable in |
| --- | --- | --- |
| `create-user` | `UserCreated` where the account was absent and was created with its credential; `Created` where the account resolved but held **no** password at all and one was set; `AlreadyPresent_NotRotated` where the account already had a password and the run did **not** request a rotation — the credential is left exactly as it was; `Rotated` where the account already had a password and the run **did** request one, which rotates `SecurityStamp` and therefore signs that account's existing sessions out. The four **partition** the operation, which is why the retired `UserAlreadyExisted` is not among them: it overlapped the last three. And `NotAttempted` where the run stopped before reaching this operation | The provisioning run, which is the only run there is; the only record on which `Failed_PublishedCredentialRefused` and `Failed_PolicyRejected` are reachable, both being credential verdicts and the credential having no record of its own ([05 §10.2](05-aspnet-core-migration-approach.md) property 4). The rotation-versus-reset decision is [06 §12.1](06-azure-hosting-recommendations.md)'s, and this catalog carries its outcomes rather than restating the decision. `Failed_UserNotFound` is **reserved with row 9's class and reachable on no run**: it presupposes an operation that resolves an account without creating it, and a provisioning run creates the absent account rather than failing on it ([05 §10.2](05-aspnet-core-migration-approach.md) property 3) |
| `create-role` | `RoleCreated` where the role was absent; `RoleAlreadyExisted` where it was present; `NotAttempted` where the run stopped before reaching this operation | Provisioning only |
| `add-membership` | `MembershipAdded`; `MembershipAlreadyPresent`; `NotAttempted` where the run stopped before reaching this operation; and, **reserved with row 9's class**, `MembershipRevoked` and `MembershipNotHeld` | The first three on the provisioning run. The reserved two are reachable on no run this plan produces — both presuppose a removal the command never performs — and are held so that a future revoke capability lands on defined outcomes rather than new ones |
| `run` | `RunSucceeded` where every operation the run reached completed. On a failed run the run record carries the **same** failure outcome the failing operation recorded, so the invocation is readable from its run record without joining the other three; on a rejected invocation it carries `Failed_ArgumentRejected` | Every invocation, without exception — this is the fourth record, and it is the one record that exists even where no operation ran |

**`NotAttempted` is a recorded state, not an absent record**, which is what keeps the count at four on
every path: all three operation records of a **rejected** invocation carry it, and so does any operation
that follows one that failed. Reading "no operation ran" off three present records is an assertion; reading
it off three missing ones would be an inference from silence, and silence is also what a lost record looks
like.

**The rejection contract — because an audit record that requires a validated value on the outcome that
exists for invalid values cannot be written.** This is the one place in the catalog where the general
rules of §6.8.1's fourth convention collide with each other: that convention requires the actor and the
target username to be *validated* and requires them *present* on a provisioning record, and it
simultaneously forbids a rejection record from echoing the value that was rejected. On the argument-
rejection branch at least one of those values is, by construction, the invalid one — so "validated,
present, and not the submitted value" has no satisfiable interpretation, and an implementer meeting it
literally writes the submitted value into an audit field, which is CWE-117 arriving through the contract
rather than through a bug. The contract below removes the collision structurally.

**Rule 1 — cardinality is the same four records every invocation emits, and no operation runs.** All input
validation completes
**before** the first store call of the run. A rejected invocation therefore emits **exactly four**
`PROV-6001` records — the three operation records at `NotAttempted`, in
[05 §10.2](05-aspnet-core-migration-approach.md) property 4's fixed order, and the **run** record at
`Failed_ArgumentRejected` — and performs **zero** operations: no role check, no user
lookup, no credential write, no membership change — so no **fifth** `PROV-6001`, and never an `AUTHZ-3001`
or `AUTHZ-3002`, can accompany them. The command exits non-zero. This holds however many fields are
invalid: one invocation, four records, exactly one of which names the first failing field.

*An earlier form of this rule made the rejected path emit **exactly one** record. That contradicted the
four-record invariant [05 §10.2](05-aspnet-core-migration-approach.md) property 4 states for **every**
path, and it did so in the direction that costs the reader most: a consumer counting records would have
had to special-case the one path on which nothing is supposed to have happened, and an implementation
satisfying the earlier rule would have emitted a record set no other invocation produces. The rule is
restated here as the invariant, and the count the tests assert is four.*

**Rule 2 — the validation order is fixed, so which field is reported is not an implementation choice.**
Validation stops at the **first** failing field, and the order is: (1) the actor side — platform
attestation is read, then the supplied `MUSICSTORE_AUDIT_ACTOR` value is checked; (2) the target
username; (3) the role
name; (4) the supplied credential's presence. The run record reports that first failure and nothing
about any later field, which also means an invocation with two invalid values yields one deterministic,
reproducible record set rather than whichever field the implementation happened to check first.

**Rule 3 — every field that would have held the rejected value holds a bounded sentinel instead, or is
declared absent where its domain admits no reservable literal.** Each sentinel is a reserved literal that
cannot collide with a legitimate value, and the reason it cannot is stated rather than assumed; the one
field whose domain admits no such literal — the target username, whose every legal value is a possible
account name — holds **typed absence** instead, which the table states and justifies in place of a
reservation:

| Field | Value on `Failed_ArgumentRejected` | Why it cannot collide |
| --- | --- | --- |
| `Actor` | the platform attestation where one exists — an attested identity is unaffected by a bad supplied value — otherwise the reserved literal **`unattributed`** | The literal is inside the actor character set, so it cannot be excluded by that check alone. It is therefore reserved **by declaration** and the command **refuses `unattributed` as a supplied `MUSICSTORE_AUDIT_ACTOR` value**; that refusal is what makes the reservation real rather than a hope, and it is one of the five `RejectedField = actor` cases the tests drive |
| `SuppliedActorClaim` | **`rejected`** where a `MUSICSTORE_AUDIT_ACTOR` value was supplied and failed; the field is **absent** where none was supplied | Reserved by the same declaration: the command refuses `rejected` as a supplied `MUSICSTORE_AUDIT_ACTOR` value, exactly as it refuses `unattributed` |
| `ActorSource` | `platformMetadata` or `environmentVariable` exactly as on any other outcome — the field names the channel the `Actor` came from, and on a rejection with no attestation it is `environmentVariable` even though the claim itself was refused, because that is still the channel that produced the sentinel | Its two literals are already closed |
| `Target` | **Declared absent — always, on this outcome.** Not a literal, not an empty string and not a null: the field is **not present** in the record, and its absence is part of the contract rather than a missing value | **No literal can be reserved in this domain, which is why typed absence replaces one.** The user-name set [05 §6.1](05-aspnet-core-migration-approach.md) owns is letters and digits, so **every** candidate sentinel — `none`, `rejected`, `unknown` — is itself a legal user name, and the Identity migration preserves source user names exactly ([05 §5.5](05-aspnet-core-migration-approach.md)). Reserving one would therefore have made an account that legitimately bears that name **unmanageable by the audited path**, pushing the operator to the hand-edit this whole section exists to eliminate — so **the command reserves no user name and refuses none on account of its text alone**. Absence is representable because the record is a structured document and rule 3 already uses declared absence for `SuppliedActorClaim` and `RoleName`, and it is unambiguous because the reason is carried by the two closed codes of rule 4: `RejectedField = targetUsername` says a target was supplied and failed, and any other `RejectedField` value says validation stopped before the target was ever examined — which is also why an unexamined supplied value must not be written here |
| `RoleName` | **`rejected`** where a role name was supplied and failed the closed-set check; **absent** where none was supplied | The closed role set is exactly one member, `Administrator`, so `rejected` is provably outside it |
| `Operation` | **`run`** — the field names the record, and this outcome is the **run** record's, no operation having run; the invocation's other three records carry their own `create-user`, `create-role` and `add-membership` values at `NotAttempted` | Which field failed is `RejectedField` below, **not** `Operation`. *An earlier form of this cell used `Operation` to name the failing field — `user`, `role` or `credential` — which duplicated `RejectedField` in a vocabulary [05 §10.2](05-aspnet-core-migration-approach.md) does not define, and left the field with no value at all where the actor was what failed* |
| `FailureCategory` | `Failed_ArgumentRejected`, the outcome value itself | Already closed |

The table states the rule for the **run** record, which is the one of the four that carries
`Failed_ArgumentRejected`. **The same rules bind the invocation's three `NotAttempted` operation records
field for field** — an invalid value is no more writable on a record for an operation that never ran than
on the run record, so those three carry the identical sentinels and the identical declared absences, and
`RejectedField` and `RejectionRule` appear on the run record alone because the rejection is the run's.

**Rule 4 — the record carries two bounded codes in place of the value, and both sets are closed.**

| Field | Closed set | Note |
| --- | --- | --- |
| `RejectedField` | `actor`, `suppliedActorClaim`, `targetUsername`, `roleName`, `credential` — **five** values | Exactly one is present, naming the field the fixed order stopped at |
| `RejectionRule` | `Missing`, `DisallowedCharacter`, `ControlCharacterOrLineBreak`, `LengthExceeded`, `NotAMemberOfClosedSet` — **five** values | Exactly one is present, naming the rule that fired. `credential` pairs **only** with `Missing`: the credential is never character-checked or length-checked by the command, so no rule that quantifies a secret can ever be reported against it, and the published-credential case is its own outcome, `Failed_PublishedCredentialRefused` |

**Rule 5 — nothing derived from the rejected value is ever written, and this is absolute.** The record
carries the two codes above and nothing else about the value: **not** the value, not a truncation of it,
not a prefix or a first character, not its length or a bucketed length, not a hash or an HMAC of it, not
a character-class summary, and not an escaped or encoded rendering. The distinction the contract draws is
between the **rule that fired**, which is a bounded code chosen from a five-member set fixed above and is
what an operator needs in order to correct the invocation, and any **property of the value**, which is
not. `RejectionRule = LengthExceeded` is the former: it names the check, and it is reportable precisely
because the check's bound is a published constant of the contract rather than a measurement of the input.

**Rule 6 — the tests that make this enforceable rather than aspirational**, added to §6.8.1.1's criterion
6. A rejection is driven for **each** of the five `RejectedField` values, and each asserts four things
structurally, on the parsed records: the invocation's `PROV-6001` records are **exactly four** — the three
operation records at `NotAttempted` in the fixed order of
[05 §10.2](05-aspnet-core-migration-approach.md) property 4, and the run record at
`Failed_ArgumentRejected`; **no fifth** `PROV-6001`, and no `AUTHZ-3001` or `AUTHZ-3002`, was emitted and no store
write occurred; every field of Rule 3 holds its declared sentinel or is declared absent, on all four
records; and — the
assertion that closes CWE-117 — the submitted value **does not appear as a substring of the serialized
record**, in any field, in any encoding. The last one is deliberately a whole-record substring assertion
rather than a per-field one, because a per-field assertion passes on the exact defect this contract
exists to prevent: a value that leaked into a field nobody thought to check. A test driven with a value
containing a line break and a value exceeding the 128-character bound covers the two forms the fourth
convention refuses to repair.

**Two representability rules bind every row, and they are stated because three rows previously broke
them.** An audit contract whose actor or target cannot exist on one of its own outcomes is not
implementable — the emitting code has nothing to put in the field, and whoever writes it will invent a
value, most likely the submitted one.

- **Every actor and target value must exist on every outcome the row lists.** Where a failure outcome
  means the entity was never created, the row names the sentinel explicitly: `anonymous` or `none`, both
  drawn from the bounded-literal convention above. Rows 5 and 11 carry those sentinels for exactly
  this reason — and neither collides, because both targets are `UserId` values, a domain no such literal
  belongs to; rows 12 and 13 do not need one, because their `AlbumId` is the route value the request
  supplied and exists whether or not the row was found. **Row 16 is the harder case, and it is closed by
  typed absence rather than by a second sentinel.** On `Failed_ArgumentRejected` — and on the three
  `NotAttempted` records of the same invocation, which the rejection contract's rule 3 binds identically —
  the missing target is not
  one that was never created but one that **must not be written** — and, unlike rows 5 and 11, its domain
  admits no reservable literal at all, because every letters-and-digits string is a legal user name that
  the migration may have carried forward from source. *An earlier form of this bullet reserved `rejected`
  and `none` here and had the command refuse them as supplied user names. That is withdrawn: it made two
  source-valid accounts unmanageable through the audited command, which contradicts the user-name
  preservation [05 §5.5](05-aspnet-core-migration-approach.md) requires and trades a real gap for a
  cosmetic one.* The field is **declared absent** on those records, the rejection contract above states
  it per field, and the two closed code sets say **why** it is absent — which is more than a sentinel
  conveyed.
- **Every outcome value is a member of a closed set, and no outcome admits free-form text.** A failure
  outcome carries a **category code** from this table and nothing more. Identity's `IdentityResult`
  error descriptions, provider messages and exception text are diagnostic output, not audit fields: they
  go to the ordinary log or the command's diagnostic stream, and never into a `PROV-6001` or `ACCT-2001`
  record. **The diagnostic channel is not a looser channel**, and this document does not imply one: it is
  bounded by [06 §9.2](06-azure-hosting-recommendations.md)'s controls 3 and 4 — the application never
  passes a caught exception object to `ILogger` at all, and what it records instead is the **sanitized
  exception record whose closed field list is [06 §9.6.2](06-azure-hosting-recommendations.md)'s shape
  R2** — cited by shape rather than by field count, per the drift rule in this sub-section's preamble. The
  property this audit rule leans on is not R2's length but its closure: **types only and never messages**,
  so no exception message, no inner message, no `ToString()` and no serialized exception object reaches
  that channel however many fields the shape comes to carry — together with
  [05 §8.3](05-aspnet-core-migration-approach.md)'s handling of the same values on the response side. So
  the value that is excluded from an audit field is not thereby admitted to a log line; it is excluded
  from both, by two documents, and the audit rule here is the narrower of the two rather than the only
  one. Without this rule the closed field lists above are
  unenforceable, because "the error description" is a channel through which any value at all can reach
  the sink.

#### 6.8.1.1 The producer map — which component emits each class, and in which workstream it can first exist

**Sixteen classes do not have one producer, and treating them as if they did produces a gate nobody can
pass.** So the catalog names, per class, the component that emits it, the interception point that makes
emission possible, and the workstream in which that component first exists.
[03](03-modernization-roadmap.md) scopes its emission and collection gates against this column rather than
against the whole catalog.

**The split, counted, because every gate downstream is written against these numbers.** The ported
web application produces **thirteen** of the sixteen classes:

`AUTH-1001`, `AUTH-1002`, `AUTH-1003`, `AUTH-1004` (**4**) + `ACCT-2001`, `ACCT-2002`, `ACCT-2003`
(**3**) + `AUTHZ-3003` (**1**) + `ADMIN-4001`, `ADMIN-4002`, `ADMIN-4003` (**3**) + `ORDER-5001`,
`ORDER-5002` (**2**) = **13**.

**Two** are produced by tooling rather than by the application: `PROV-6001` from the provisioning
command only, and `AUTHZ-3001` from the provisioning command **and** the Identity data migration — two
producers for one class. **One is produced by nothing this plan builds**: `AUTHZ-3002`, whose row below
carries no producer at all. So the split is **13 + 2 + 1 = 16**, and it is written that way rather than as
13 + 3 because the third number is not a producer count. The two tool-produced classes exist in tooling
the port does not contain, which is why a gate demanding all sixteen from the port would be failed by a
correct implementation; the reserved class is demanded of nothing, and the note after the table states
what that leaves open and who owns it.

| Classes | Producer | Interception point | First exists in |
| --- | --- | --- | --- |
| `AUTH-1001`, `AUTH-1002`, `AUTH-1004` | The ported web application's account controller | The sign-in sequence and sign-out action of [05 §4.3](05-aspnet-core-migration-approach.md) and [§6.2](05-aspnet-core-migration-approach.md) — explicit emission at each outcome branch, not a framework event | The port |
| `AUTH-1003` | The ported web application, but **not** from the controller | **Seam 3 below** — an override of the virtual `UserManager<ApplicationUser>.AccessFailedAsync`. The lockout *transition* is performed inside Identity's own call and cannot be read atomically from the controller either side of it | The port |
| `ACCT-2001`, `ACCT-2003` | The ported web application's account controller | The registration sequence of [05 §4.3](05-aspnet-core-migration-approach.md) and the change-password action, at each `IdentityResult` branch. `ACCT-2001`'s two failure outcomes are the branches Identity itself distinguishes: a duplicate user name is `DuplicateUserName`, **never** `ValidationFailed`, which is reserved for a bound or policy rejection | The port |
| `ACCT-2002` | The ported web application's account controller, for a user-initiated change; **seam 1 below** for the `RehashedOnSignIn` case | The change-password action's success branch; and, for a rehash, **two artifacts** — a decorating `IPasswordHasher<ApplicationUser>` that only *observes* the rehash advice, and an override of the `protected virtual UserManager<ApplicationUser>.UpdateUserAsync` that emits only on a **successful** store result. The rehash happens *inside* `CheckPasswordSignInAsync`, which is invisible to the caller, and the update result it depends on is **discarded** by `CheckPasswordAsync`, so the hasher alone cannot tell a persisted rehash from a failed one | The port |
| `AUTHZ-3003` | The ported web application | **Seam 2 below** — a custom `IAuthorizationMiddlewareResultHandler` **that delegates to the framework's own handler**, registered in the composition root. This is the one class with no natural emission site in application code: an authorization failure is decided by middleware and never reaches the action, so a controller cannot log it and a global action filter does not run. Naming the mechanism matters — an implementation that instead logs from an access-denied page misses every non-`GET` denial and every API-shaped 403 | The port |
| `ADMIN-4001`, `ADMIN-4002`, `ADMIN-4003` | The ported web application's administration controller | The input-model actions of [05 §8.11](05-aspnet-core-migration-approach.md), at each branch | The port |
| `ORDER-5001`, `ORDER-5002` | The ported web application's checkout controller | The ordered write path of [05 §8.12](05-aspnet-core-migration-approach.md), including the failure table that replaces the source's bare `catch` | The port |
| `AUTHZ-3001` — role granted | **Two producers, and neither is the web application.** The provisioning command for an operator grant; the Identity data migration for each assignment it loads | The command's membership operation; the migration's assignment load, one record per assignment | The provisioning tool for the first; the Identity migration tooling for the second |
| `AUTHZ-3002` — role revoked | **None. The identifier is reserved and nothing in this plan emits it** — an earlier form of this row named "the provisioning command's revoke mode", which the command does not have | **None.** There is no interception point, because there is no operation to intercept: [05 §10.2](05-aspnet-core-migration-approach.md) property 3 states the command never deletes or demotes, its verb table is closed with no revoke verb, and the `admin` verb's switch set is closed at four. The earlier form cited that section's "property 3a", which it does not define | **Nowhere yet** — first exists in whatever capability a future decision adds, per the note below |
| `PROV-6001` | The provisioning command | **Four records per invocation, invariantly** — one for each of the three operations [05 §10.2](05-aspnet-core-migration-approach.md) property 3 checks and one for the run, in that section's fixed order and on every path including a rejected invocation — written as one JSON object per line to the command's **standard output** by a **dedicated serializer and not through `ILogger`**, because the pre-host rejection path has no logger to resolve, then selected by event name and count and lodged as one append-only blob by the four-hop route of [06 §9.5.1](06-azure-hosting-recommendations.md) | The provisioning tool |

> **`AUTHZ-3002` is a reserved identifier with no producer, and this note is the whole of its status.**
> An earlier form of this catalog attributed the class to "the provisioning command's revoke mode",
> specified in a "property 3a" of [05 §10.2](05-aspnet-core-migration-approach.md). **That property does
> not exist and that mode does not exist.**
> [05 §10.2](05-aspnet-core-migration-approach.md) defines properties 1, 1a, 1b, 2, 2a, 2b, 3, 4 and 5;
> its property 3 states the command never deletes or demotes anything; its verb table is closed and holds
> no revoke verb; and the `admin` verb's accepted switch set is closed at four, none of which asks for a
> revocation. An event class whose producer is a property number nobody wrote is the defect this note
> exists to end, and it is ended the only way the evidence allows: **the attribution is withdrawn, and no
> substitute producer is named, because no artifact in this plan withdraws a role.**
>
> **Reclassifying it onto an existing producer was considered and rejected as false.** The provisioning
> command is the only candidate, and property 3 rules it out in as many words; the Identity data migration
> loads assignments and removes none; and the ported web application has no role-management surface at
> all — §6.8.1.1 assigns the grant class no application producer for the same reason. Naming any of them
> would move the defect rather than fix it.
>
> **Adding the capability here was also rejected, and the reason is a contract rather than a preference.**
> A revoke mode would need a fifth `admin` switch and a fifth audited operation, and both sets are pinned
> closed: [05 §10.2](05-aspnet-core-migration-approach.md) property 1b rejects any switch outside the
> accepted four before a host is built, and its property 4 fixes **exactly four records per invocation**
> with a closed operation sequence, which a revoke run's operations would contradict. Specifying one from
> this document would silently break two contracts that other deliverables and their release-time
> assertions are written against.
>
> **Striking the class was rejected too, because the identifier is worth keeping and the gap is worth
> stating.** Deleting the row would renumber nothing and cost nothing structurally, but it would erase a
> real finding: after F-09-05's remediation an administrator's credential is rotatable and their
> **membership** is not, so **an operator who must remove someone's administrative access still has no
> audited way to do it** and reaches for `AspNetUserRoles` by hand — the unaudited path this entire section
> exists to eliminate. Keeping the identifier reserved records that gap where a reader of the audit
> catalog meets it, and means a future capability inherits a defined record shape — row 9's outcome,
> severity and field list — instead of inventing one and drifting from
> [06 §9.5](06-azure-hosting-recommendations.md)'s table.
>
> **So the status is exactly this, with nothing implied beyond it.** The gap is **open**, its owner is
> **security engineering**, and **no workstream carries it** — deliberately, because a workstream needs a
> specified artifact and there is none. Closing it takes a decision this document does not have the
> standing to take: it is net-new tooling capability, so it is not an approved delta in
> [05 §11.5](05-aspnet-core-migration-approach.md)'s sense either, and until that decision is taken and
> the switch and record contracts above are reopened by their owner, **nothing emits this class and no
> gate, test or count may demand that anything does.**

**What an emission establishes, and what it does not — four statements that bind every class in this
catalog, and the second of them governs.** They are stated here, once, because a catalog that requires
records is worth nothing if what "recorded" means is left to the reader. An `ILogger` call is a
**synchronous dispatch to every registered provider** and nothing more: the console provider serializes
and writes its line on its own background thread, and the platform's collection is asynchronous after
that. A synchronous physical write is therefore not a property any emission site can have, and **no claim
of one, and no ordering claim stronger than the hand-off in (a), is made anywhere in this document.**

- **(a) Ordering is a hand-off, and the hand-off is the whole of the ordering requirement.** At every
  emission site the record is **constructed and handed to `ILogger` before the effect is committed**,
  wherever that order is observable at all — before the `403` is issued, before the authentication cookie
  is issued, before the response is written — and no required record is emitted after the response has
  begun to be written. What the call's return establishes is that the record was fully constructed and
  dispatched to the registered providers, and nothing beyond that. The channel-side statement of the same
  rule is [06 §9.5](06-azure-hosting-recommendations.md)'s property 3, which this document cites rather
  than restates.
- **(b) A failure to emit never alters what the application does — this is this document's requirement and
  it governs.** Where the logging call throws, where a provider fails, and where the line is lost after
  the hand-off, the operation completes exactly as it would have with logging healthy: the same status
  code, the same redirect, the same cookie, the same response bytes, the same `IdentityResult`. Emission
  is **wrapped** at every site for that reason, and **no record in this catalog may be made a precondition
  of a business effect** — nothing is suppressed, withheld, rolled back or downgraded because a record may
  not have been written, and a logging fault is never converted into an availability fault on the sign-in,
  authorization or checkout paths. Seam 2 below states this for the response and is the statement
  [06 §9.5](06-azure-hosting-recommendations.md)'s property 4 cites and defers to; it holds identically
  for the other fifteen classes.
- **(c) Delivery is asserted out of band, never by the logging call's return.** That a class's records
  actually arrive is proven by the per-producer collection gates of
  [06 §9.5](06-azure-hosting-recommendations.md) — whose application arm is a canary security-event record
  asserted present in the workspace by its event identifier — and by closure criterion 3 below, which
  counts four producer-and-class checks. A producer whose records stop arriving is caught **there**, by
  detection, and never by a caller declining to proceed.
- **(d) A record can still be lost, and that is an accepted limitation rather than a gap with a
  mitigation.** A process that terminates between the hand-off and the platform's collection of the
  resulting line loses that record. The window is bounded only by the platform's capture cadence and by
  graceful shutdown; it is small, it is not zero, and nothing in this design closes it —
  [06 §9.5](06-azure-hosting-recommendations.md) records the same window from the hosting side and this
  document claims nothing stronger against it. The consequence for this catalog is stated rather than
  absorbed: **F-09-32's closure asserts that every class is emitted and that each producer's collection
  path works, which is strictly weaker than every individual record surviving**, and no gate, query or
  acceptance criterion here is written as though the stronger property held. There is likewise no
  write-once, append-only or otherwise immutable destination to appeal to — 06 §9.5 withdrew the one this
  sub-section's earlier form assumed, as the amendment paragraph of §6.8.1 records, and §6.8.2 carries the
  one control that the withdrawal costs — control 3, the legal hold — as an accepted gap with named owners.

**The one fail-closed requirement this document does make is a different kind of thing, and the
distinction is worth keeping visible.** The `SubjectPseudonym` key contract above requires the application
to **refuse to start** where its key record is absent or fails 06 §8.4.7's startup validation. That is
**startup configuration validation** — a deployment-time property, decided before any request is served,
and implementable exactly as written. Suppressing a completed business effect because a log line may not
have been flushed is not implementable at all, and (b) is why this document asks for the first and never
for the second.

**Three of the thirteen application-produced classes have no emission site in ordinary controller code,
and each needs a named seam rather than an instruction.** These are the three an implementation quietly
drops, because the fact each one records happens *inside* a framework call the controller cannot see the
inside of. Each seam below is a concrete artifact with a registration and a stated effect on existing
behaviour; each replaces or extends a type the target framework exposes for the purpose, and none of them
changes what a user sees.

**Seam 1 — `ACCT-2002` with `RehashedOnSignIn`: a decorating password hasher *and* a store seam, because
the hasher alone can assert a rehash that never persisted.** A migrated account signs in, its stored hash
is in an older format, and Identity rehashes and re-saves it. That write is a change to the account's
credential material and belongs in the audit trail — but it happens inside
`SignInManager.CheckPasswordSignInAsync`, which returns a sign-in result and says nothing about it.

**Why the hasher decorator is necessary but not sufficient, stated first because it is the defect this
seam exists to avoid.** The decorator's signal fires when the hasher returns
`PasswordVerificationResult.SuccessRehashNeeded` — one of that enumeration's exactly three members,
`Failed`, `Success` and `SuccessRehashNeeded`, verified against the `Microsoft.AspNetCore.App.Ref` **8.0.30**
reference pack. That result is the hasher's **advice that a rehash is needed**, and it is produced
**before** `UserManager` attempts any persistence. Worse, `UserManager.CheckPasswordAsync` then rehashes
and calls its own `UpdateUserAsync` **without inspecting the `IdentityResult` that call returns** — the
result is discarded. So an implementation that emits `ACCT-2002` on the hasher's signal alone writes an
audit record asserting that an account's credential material changed on runs where the store rejected the
write and the account still holds its legacy hash. **An audit record that asserts a write which did not
happen is worse than a missing one**, because it is the record a later investigation trusts. The rule
is therefore: `ACCT-2002` with `RehashedOnSignIn` is emitted **only on a store-observed successful hash
change**, and the seam that observes it is a second override, not the hasher.

| Part | Decision |
| --- | --- |
| **Artifact 1 — the observer** | `RehashObservingPasswordHasher : IPasswordHasher<ApplicationUser>`, a **decorator whose single hasher dependency is the *concrete* `PasswordHasher<ApplicationUser>`** — never the `IPasswordHasher<ApplicationUser>` service type the decorator is itself registered as, for the reason the registration row below states in full — and **there is one of it, not one per consumer**. [05 §4.3](05-aspnet-core-migration-approach.md) owns the composition root, writes the registration lines and registers this same artifact for its own uniform-cost purpose; **this row and the registration row below own the constructor dependency and the registration shape, and they are what [04](04-dotnet8-migration-strategy.md) and 05 cite for the mechanism**. On `VerifyHashedPassword` it delegates to the inner hasher, records that a derivation ran and whether the inner result was `PasswordVerificationResult.SuccessRehashNeeded`, and **returns the inner value unchanged**. It **never emits** — it only observes. *A subclass of `PasswordHasher<ApplicationUser>` overriding the same method is equally possible — the type is not sealed and the method is virtual in the `Microsoft.AspNetCore.App.Ref` **8.0.30** reference pack — and is deliberately **not** the specified form: it would fork this artifact into two incompatible types, and a subclass binds its own base implementation at compile time where the decorator's inner hasher is a registration that can be changed without touching this type* |
| **Artifact 2 — the store seam that emits** | `AuditingUserManager : UserManager<ApplicationUser>` — the **same subclass seam 3 introduces**, one type with two overrides — additionally overriding `protected virtual Task<IdentityResult> UpdateUserAsync(ApplicationUser user)`. That member is `protected virtual` and returns `Task<IdentityResult>` in the **8.0.30** reference pack, and it is the single point every credential write funnels through: `CheckPasswordAsync` reaches it via the equally `protected virtual` `UpdatePasswordHash`, and `ChangePasswordAsync`, `ResetPasswordAsync` and `AddPasswordAsync` reach it the same way. The override calls `base`, holds the returned `IdentityResult`, decides emission from it, and **returns it unchanged** |
| **The signal — one scoped object, one name** | **`PasswordDerivationObservation`**, the scoped per-request state [05 §4.3](05-aspnet-core-migration-approach.md) declares and whose two readers it already names: its uniform-cost top-up, and this seam's `ACCT-2002` producer. It is **one service with two consumers**, not two services — an earlier form of this section called it `PasswordRehashSignal`, which is the same object under a second name and is recorded here only so the two identifiers are not implemented as two registrations. Scoped, so it is neither static nor ambient and two concurrent sign-ins cannot see each other's value. Its members and their owners: **`DerivationRan`**, the format marker and the iteration count, all set by artifact 1 and owned by 05 §4.3 for the top-up; **`RehashObserved`**, set by artifact 1 where the inner result was `SuccessRehashNeeded`; and **`UserInitiatedCredentialChange`**, set by the change-password action before it calls `ChangePasswordAsync`. The last two are this seam's, and the second of them is the one addition this seam makes to 05's declared shape |
| **Read and reset — exactly where, and by whom** | The two audit members are **consume-once**: they are read in **one** place, inside artifact 2's `UpdateUserAsync` override, **after** `base` returns, and the read **clears** the member as part of the same read. Both are read **unconditionally at that point, before the emission decision is taken** — not only on the branch that emits — because a write that failed must still consume the observation, or a set flag would survive into the next `UpdateUserAsync` of the same request and that later, unrelated write would emit a rehash record for a rehash that never persisted. Nothing else reads or writes them. Consume-once matters because a successful sign-in can reach `UpdateUserAsync` **more than once** in one request — the rehash write, and then the reset of an accumulated access-failed count — so only the first of those reads a set flag and the rehash is audited exactly once. 05 §4.3's own members are **not** consume-once: the top-up reads `DerivationRan` on the request path before any store write, and this seam neither reads nor clears them, so the two consumers cannot interfere. Every member is at its default at the start of each request, because the object's lifetime is the request |
| **Concurrent verification of one account** | Two simultaneous sign-ins for the same legacy-hash account are two request scopes, so **two** observations are set and **two** `UpdateUserAsync` calls are attempted. Emission follows each call's **own** `IdentityResult`, and `ApplicationUser.ConcurrencyStamp` — which the Identity migration populates for every account — makes at most one of the two writes succeed. So at most **one** `ACCT-2002` with `RehashedOnSignIn` is emitted for the account, and the losing request emits the failed-update diagnostic below instead, which is the correct account of what happened to it. This class therefore needs **no** de-duplication key: unlike `AUTH-1003` in seam 3, whose multiplicity comes from delivery semantics, `ACCT-2002`'s cardinality is bounded by the store's own concurrency control, and adding a key would imply a multiplicity this design does not have |
| **Registration — four registrations, one of each artifact, and the inner hasher is one of them** | **The defect this row exists to prevent, stated before the shape, because an earlier form of this row specified it.** A decorator registered *as* `IPasswordHasher<ApplicationUser>` whose constructor parameter is also `IPasswordHasher<ApplicationUser>` is **self-referential**: the container satisfies that parameter through the same service type, which resolves to the decorator, and the resolution recurses until it fails. `Microsoft.Extensions.DependencyInjection` has **no descriptor-capturing decoration primitive**, so no registration order and no "wrap the previously registered hasher" phrasing rescues the shape — it cannot be constructed at all, and the third-party library that supplies such a primitive is a package neither [02](02-dependency-inventory.md) nor [04](04-dotnet8-migration-strategy.md) carries. **The mechanism, which is this row's to state:** the inner hasher is made **separately resolvable by its concrete type**, and the decorator depends on that concrete type. The four registrations, each written once, are (1) `PasswordDerivationObservation` as a **scoped** service; (2) **`PasswordHasher<ApplicationUser>` registered as its own concrete type, scoped** — the type is public and non-sealed and its constructor parameter is an `IOptions<PasswordHasherOptions>` in the `Microsoft.AspNetCore.App.Ref` **8.0.30** reference pack, which the container satisfies from the options Identity already configures; (3) `RehashObservingPasswordHasher` as the `IPasswordHasher<ApplicationUser>` implementation, **scoped**, taking (2) and (1) as its constructor dependencies; and (4) `AuditingUserManager` exactly once, by seam 3's `AddIdentity<ApplicationUser, IdentityRole>().AddUserManager<AuditingUserManager>()`, so every caller including `SignInManager` goes through both of that type's overrides. **The property that makes the seam constructible is that (3) does not depend on the interface at all** — no ordering can make the interface resolve to itself — and the placement of the lines relative to `AddIdentity`, which decides only *which* descriptor the interface resolves to, is [05 §4.3](05-aspnet-core-migration-approach.md)'s to fix and is fixed there. **No fifth registration exists**: one observation object, one inner hasher, one decorator, one user manager. There is no separate rehash-signal service, no second decorator and no second user manager, and an implementation that adds one has two objects where the contract has one; an implementation that omits (2) has a shape the container cannot build, which is a startup failure rather than a missing audit record |
| **Emission — the exact contract** | Inside the `UpdateUserAsync` override, after `base` returns: emit **one** `ACCT-2002` at `Success` with `RehashedOnSignIn = true` **if and only if** (a) the returned `IdentityResult.Succeeded` is **true** *and* (b) the scoped `RehashObserved` flag is set *and* (c) `UserInitiatedCredentialChange` is **not** set. Any other combination emits **no** `ACCT-2002` from this seam. The record accompanies the sign-in's `AUTH-1001`, which the controller emits as before. A user-initiated change is unaffected: condition (c) suppresses the rehash record — the hasher also reports `SuccessRehashNeeded` while verifying the *current* password on a legacy-hash account, and without (c) one credential change would write two `ACCT-2002` records — and the change-password action emits its own `ACCT-2002` at `Success` on the `IdentityResult` success branch, **without** `RehashedOnSignIn`. **One credential change, one record, in every combination** |
| **Failed update — a distinct diagnostic, not an `ACCT-2002`** | Where (b) holds and `IdentityResult.Succeeded` is **false**, the store did not persist the rehash and the account still holds its legacy hash. The seam emits **no** `ACCT-2002` and instead **one diagnostic record on the ordinary application diagnostic channel** — not into this catalog, which stays at **sixteen** classes — at **Warning**, carrying the `UserId`, the operation label `credential-rehash`, the request correlation identifier and Identity's error **codes**. Never the error descriptions, never the candidate or stored hash: the permitted-content rules of [06 §9.2](06-azure-hosting-recommendations.md) and [05 §8.3](05-aspnet-core-migration-approach.md) govern it exactly as they govern any diagnostic. It is a **diagnostic, not an audit fact**, because nothing security-relevant changed — which is the whole point of separating the two |
| **Behaviour impact** | **None.** Artifact 1 returns the inner verification result untouched and artifact 2 returns the base `IdentityResult` untouched, so verification, rehash, sign-in and every other credential write behave exactly as the framework's own types. Emission is wrapped so that a failure to write a record cannot alter or block the returned result — statement **(b)** of the emission contract above, which seam 2 states in the same words for the response |
| **Rejected alternative 1** | Emitting from the hasher decorator, or from the controller reading its signal after sign-in. Both observe the hasher's **advice** rather than the store's **outcome**, and `CheckPasswordAsync` discards the update result, so both can assert a rehash that never persisted. This is exactly the defect the seam replaces, and it is recorded here so the cheaper form is not reintroduced as a simplification |
| **Rejected alternative 2** | Reading `user.PasswordHash` before the call and comparing it after. It needs a second store read, it reads an entity the sign-in path may already have updated in the tracker, and it **cannot distinguish a rehash from a concurrent password change** on the same account — so it would report a credential change that the audited actor did not make |
| **Rejected alternative 3 — and it is the one an implementer reaches by accident** | Building the per-request state twice, once per consumer, because two deliverables named it differently — a rehash signal for the audit record and a derivation observation for the uniform-cost top-up. It compiles, both registrations resolve, and it fails in the only way that matters: artifact 1 writes to one of them and the other is never set, so either the top-up pads every request as though no derivation ran or `ACCT-2002` never carries `RehashedOnSignIn`, with nothing failing loudly in either case. The contract is **one scoped object with one name**, stated in the signal row above, and its two consumers are enumerated there and in [05 §4.3](05-aspnet-core-migration-approach.md) |

**Seam 2 — `AUTHZ-3003`: a result handler that delegates.** An authorization failure is decided by the
authorization middleware and never reaches an action, so the emission point is the middleware's result
handler. The trap is that replacing the handler replaces the **response** for every denial, not only the
logging, so a naive implementation changes challenge and forbid behaviour across the whole application
while intending only to add a log record.

| Part | Decision |
| --- | --- |
| **Artifact** | `AuditingAuthorizationMiddlewareResultHandler : IAuthorizationMiddlewareResultHandler`, holding an instance of the framework's own `Microsoft.AspNetCore.Authorization.Policy.AuthorizationMiddlewareResultHandler` — a public, non-sealed type with a public parameterless constructor — as its inner handler |
| **Registration** | `services.AddSingleton<IAuthorizationMiddlewareResultHandler, AuditingAuthorizationMiddlewareResultHandler>();`. The framework registers its default with `TryAddSingleton`, so an explicit registration takes precedence; the inner handler is constructed directly rather than resolved, because the interface now resolves to the decorator |
| **Emission** | Where the authorization result did **not** succeed — challenged or forbidden — one `AUTHZ-3003` record at `Denied`, carrying `DenialKind`, `RequiredPolicy`, and `ControllerName` and `ActionName` from the endpoint's `ControllerActionDescriptor` metadata (`ControllerName` and `ActionName` are declared properties of that type). The actor is the request's `UserId` claim, or the `anonymous` literal §6.8.1 defines. The two derived fields are contracted in the two rows below, because neither can be taken from the obvious place |
| **`DenialKind` — the distinction the record exists to make, and it comes from the result, not the policy** | Two closed values, `Challenged` and `Forbidden`, read from the `PolicyAuthorizationResult` argument's `Challenged` and `Forbidden` properties. Without this field `AUTHZ-3003` cannot separate **"the caller was not authenticated"** from **"the caller was authenticated and not permitted"** — which is the single most useful distinction in an authorization trail, and the two are also the two different *responses* the inner handler produces, so a record that conflates them cannot be reconciled against the response it accompanied. It is a field rather than an outcome value: class 10's `Outcome` remains the single value `Denied`, so §6.8.1's outcome count is unchanged |
| **`RequiredPolicy` — derived from endpoint metadata, because the policy argument cannot supply it** | **`AuthorizationPolicy` has no name.** The type declares exactly two public members, `Requirements` and `AuthenticationSchemes`, and a name is bound only where `AuthorizationOptions.AddPolicy(string, …)` registered one — the policy *object* handed to the result handler does not carry it. An earlier form of this row said `RequiredPolicy` came "from the policy argument", which is not implementable: there is nothing to read, and the nearest available substitute, a `ToString()` of the policy or of its requirements, yields framework type names that are neither stable nor meaningful and are exactly the free-form value §6.8.1's third convention forbids. The field is therefore derived from the **endpoint**, by the four-step rule below it, and validated against a closed set before it is written |
| **Delegation is mandatory, on every path** | The decorator **always** invokes the inner handler exactly once, for success and failure alike, and returns what it returns. Without that, every denial's status code, redirect to the login path and access-denied behaviour become whatever the decorator happens to do — a change to the application's authorization behaviour introduced by an audit requirement. Emission is wrapped so that a failure to write a record cannot alter or block the response |
| **Emission never alters the response — the governing statement of this document's audit contract, and the one [06 §9.5](06-azure-hosting-recommendations.md)'s property 4 cites** | The record is constructed and handed to `ILogger` **before** the inner handler produces the response, wherever that order is observable, and the emission is **wrapped**: where the logging call throws, where a provider fails, and where the line is lost after the hand-off, the denial's status code, `Location` header and body are exactly what the framework handler produced. **A failure to emit never suppresses, delays, downgrades or otherwise changes a denial or any other authorization outcome, and no `AUTHZ-3003` record is a precondition of any response.** That is statement **(b)** of the emission contract above — this document's requirement — and it **governs** wherever a stronger property is proposed on either side of the ownership line. What the hand-off does **not** establish is a physical write or platform collection: delivery is asserted by statement **(c)**'s per-producer gates and canary record, and statement **(d)**'s loss window between hand-off and collection is carried as an accepted limitation rather than closed by anything here |
| **Behaviour impact** | **None**, by construction: the response is produced by the framework handler in both cases. That is the property the coverage assertion checks — a denial's status and location must be byte-identical with the decorator registered and unregistered |

**The `RequiredPolicy` derivation, in four ordered steps, so the value is deterministic and bounded rather
than best-effort.** The handler receives the `HttpContext`, so the endpoint is available to it; the two
APIs the steps below name were confirmed against the `Microsoft.AspNetCore.App` **8.0.30** reference
assemblies — `HttpContext.GetEndpoint()` and `EndpointMetadataCollection.GetOrderedMetadata<T>()`, which
returns an `IReadOnlyList<T>` — and `IAuthorizeData` declares exactly three string properties, `Policy`,
`Roles` and `AuthenticationSchemes`.

1. **Read the endpoint's authorization metadata in metadata order:**
   `context.GetEndpoint()?.Metadata.GetOrderedMetadata<IAuthorizeData>()`. This is the *declared*
   requirement — what the route asked for — which is what an operator reading the trail needs, and it is
   the only place a policy **name** survives to.
2. **For each entry, take its `Policy` where that string is non-empty.** Those values are bounded by
   construction: they are exactly the names passed to `AuthorizationOptions.AddPolicy(string, …)` in the
   composition root, which is a closed list — the authorization map
   [05 §6.4](05-aspnet-core-migration-approach.md) fixes. Multiple entries are joined in metadata order
   with a single fixed separator, so the field is stable for a given endpoint rather than order-dependent
   on a dictionary.
3. **Where an entry declares roles instead of a policy** — the shape the source's only authorization
   attribute uses [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:12] — the field records the
   role form: the fixed prefix `roles:` followed by the entry's `Roles` value. `IAuthorizeData.Roles` is a
   comma-separated **string**, and it is written **only** after being split and matched against the closed
   role set, which has one member; an unmatched token is dropped and step 4 applies. That is what stops a
   metadata string reaching the field unchecked.
4. **Three reserved literals close the remaining cases, and nothing else may be written.** The first is the
   one an earlier form of this derivation omitted, and it is the **most frequently produced value in this
   application**, so its absence would have left the commonest denial with an empty audit field.
   **`default`** where the endpoint carries an `IAuthorizeData` entry that declares **neither** a policy
   **nor** roles — a bare `[Authorize]`, which is the shape of the source's class-level attribute on
   `AccountController` [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:15] and the shape
   [05 §6.4](05-aspnet-core-migration-approach.md)'s map carries forward on every authenticated non-admin
   surface. Steps 2 and 3 collect nothing from such an entry, and the framework applies
   `AuthorizationOptions.DefaultPolicy` to it, so `default` names exactly what was required: an
   authenticated identity and nothing further. **`unnamed`** where a value was collected but is not a member
   of the closed set. And **`fallback`** where the endpoint carries **no** `IAuthorizeData` at all — the
   denial `AuthorizationOptions.FallbackPolicy` produces. That third case is **unreachable in the deployed
   configuration**, because [05 §6.4](05-aspnet-core-migration-approach.md)'s authorization map configures
   no fallback policy and this document does not require one: a fallback policy demanding an authenticated
   identity would deny the anonymous catalogue, which is a user-visible behaviour change with no finding
   behind it. The literal is retained for exactly one reason — so that if a fallback policy is ever added,
   its denials write a named value instead of an empty field — and the coverage below exercises it
   accordingly rather than pretending the deployed application produces it. The closed set is therefore the
   registered policy names, the role set, and these **three** literals. **The validation is at emission, not at design time**, so a
   later change to the authorization map that this catalog has not caught up with degrades the field to
   `unnamed` and writes a separate diagnostic naming the endpoint, rather than widening an audit field into
   a free-form one. `AuthorizationPolicy` itself is never rendered — no `ToString()` of the policy, of its
   `Requirements` collection or of any requirement instance appears in this field in any circumstance.

*Rejected alternative — projecting the policy's `Requirements` to their type names.* It is deterministic,
and it is available on the argument the handler already has, which is why it is tempting. It is rejected on
two grounds. It answers the wrong question: `RolesAuthorizationRequirement` tells an operator that a role
was required and not **which** role, while `DenyAnonymousAuthorizationRequirement` tells them only that
authentication was required — and step 3 above supplies exactly what those omit. And its value set is a set
of **framework** type names, so it grows and changes with framework internals rather than with this
application's authorization map, which is the opposite of the bound §6.8.1's third convention asks for.

**The coverage assertion is two denials, not one, because the two paths differ in the framework and in the
record.** `PolicyAuthorizationResult` exposes `Challenged` and `Forbidden` as separate properties, and the
inner handler produces a different response for each: a challenge invokes the authentication scheme's
challenge — for cookie authentication, a redirect to the configured login path carrying the return URL — and
a forbid produces the access-denied response. A suite that exercises one path proves nothing about the
other, and a decorator that mishandles the branch it did not test changes that path's behaviour silently.
Five cases, because step 4's three literals are three separate outcomes and two of them are not produced by
the administration route the first two cases use:

- **The challenge path.** An **unauthenticated** request to an administration route asserts exactly one
  `AUTHZ-3003` at `Denied` with `DenialKind = Challenged`, actor `anonymous`, `RequiredPolicy` at the value
  the derivation above produces for that endpoint, and `ControllerName`/`ActionName` from the descriptor —
  and the response **byte-identical** to the same request with the decorator unregistered, status and
  `Location` included.
- **The forbid path.** An **authenticated non-administrator** request to the same route asserts one record
  with `DenialKind = Forbidden` and the actor's `UserId` rather than `anonymous`, with the same
  byte-identical-response assertion. The two records must differ in exactly those two fields and agree in
  the rest, which is the assertion that catches a `DenialKind` read from the wrong property.
- **A success.** An **administrator** request to the same route asserts the inner handler ran and that
  **no** `AUTHZ-3003` was emitted. "Where the result did not succeed" is only testable against a case that
  succeeds.
- **The default-policy case.** An **unauthenticated** request to an endpoint carrying a **bare
  `[Authorize]`** — no policy and no roles, the application's commonest authorization shape — asserts one
  record with `DenialKind = Challenged` and **`RequiredPolicy = default`**, and specifically **not** an empty
  field, not `unnamed` and not the administration route's `roles:` form. This case is separate from the
  challenge case above because that one exercises a role-declaring endpoint, so it proves nothing about the
  entry from which steps 2 and 3 collect nothing.
- **The fallback case, exercised where it exists and asserted absent where it does not.** Two assertions,
  because the deployed application configures no fallback policy. **(i)** Against the deployed
  configuration, `AuthorizationOptions.FallbackPolicy` is asserted **null** — a security assertion in its
  own right, since a non-null fallback would deny the anonymous catalogue and the suite would report that as
  a browse regression somewhere far from its cause. **(ii)** In a **test-only host** that registers a
  fallback policy, a denial on an endpoint carrying no `IAuthorizeData` asserts `RequiredPolicy = fallback`,
  which is what proves the literal is produced rather than merely reserved. Asserting (ii) against the
  deployed configuration is not possible, and an implementation that made it possible would have changed the
  application's authorization posture to satisfy a test.

**Seam 3 — `AUTH-1003`: the lockout transition, and the race that a before-and-after read does not
solve.** `AUTH-1002` at `AccountLockedOut` records *an attempt against a locked account*; `AUTH-1003`
records *the account becoming locked*. Reading lockout state before `CheckPasswordSignInAsync` and again
after is the obvious implementation and it is **not** atomic: two failed attempts arriving together can
each observe "not locked before, locked after" and emit two `AUTH-1003` records for one transition — and
under a different interleaving both can observe "already locked" and emit none.

| Part | Decision |
| --- | --- |
| **Artifact** | `AuditingUserManager : UserManager<ApplicationUser>`, overriding the virtual `AccessFailedAsync`. This is the call in which Identity increments `AccessFailedCount` and, at the threshold, sets `LockoutEnd` — so the transition is inside it, and nothing outside it can observe the transition atomically |
| **Registration** | Registered as the Identity user manager through `AddIdentity<ApplicationUser, IdentityRole>().AddUserManager<AuditingUserManager>()`, so every caller — `SignInManager` included — goes through it |
| **Emission** | Inside the override: capture the lockout state, call `base.AccessFailedAsync`, and emit `AUTH-1003` **only** where the returned `IdentityResult` succeeded *and* the account was not locked before the call *and* is locked after it. A failed result means the store did not persist the change, so it emits nothing. The record carries the post-call `LockoutEndUtc`, which with the actor's `UserId` forms the de-duplication key of the next row |
| **Delivery semantics, and the durable de-duplication key** | **Delivery is at-least-once, not exactly-once, and this document does not claim exactly-once.** The store's own optimistic-concurrency token makes a losing concurrent update return a failed `IdentityResult`, which suppresses that caller's record, but that is not asserted as an exactly-once guarantee across every store configuration, and **no outbox or atomic log-and-store write is specified here**. What is specified instead is a **durable de-duplication key that every physical record carries**: `{UserId}\|{LockoutEndUtc:o}` — the actor's `UserId`, a literal pipe, and the record's `LockoutEndUtc` value formatted with the round-trip (`o`) specifier. **Two physical `AUTH-1003` records with equal keys are one logical event**, and a reader — a query, an export or a test — collapses on **exact key equality**, not on a time window. The key is computed only from values the record already carries, so it needs no coordination between producers, survives replay and retry, and is computable by any reader of the sink |
| **Coverage assertion, matched to that choice** | Two tests, not one. A **sequential** test drives the threshold and asserts exactly one `AUTH-1003`, then asserts that a further attempt while locked produces `AUTH-1002` at `AccountLockedOut` and **no** second `AUTH-1003`. A **concurrent** test fires simultaneous failed attempts across the threshold and asserts **at least one** `AUTH-1003` and that every `AUTH-1003` record for that account collapses, on exact equality of `{UserId}\|{LockoutEndUtc:o}`, to **one logical event**. An assertion of exactly one *physical* record under concurrency would be asserting a property this design does not claim |
| **Behaviour impact** | **None** — the override returns the base result, and lockout continues to be decided entirely by Identity's own policy values ([05 §6.1](05-aspnet-core-migration-approach.md)). Emission is wrapped here too, so a failure to write the `AUTH-1003` record neither blocks nor changes the lockout transition, exactly as statement **(b)** of the emission contract above requires |

**Acceptance criteria for F-09-32's closure**, so that "logging was added" is not the standard:

1. **Every one of the fifteen *produced* classes emitted**, at the identifier and severity above, **by the
   producer §6.8.1.1 assigns it** — **thirteen** from the ported application; `PROV-6001` from the
   provisioning command alone; and `AUTHZ-3001` from the provisioning command **and** the
   Identity data migration, the one class with two producers. **The sixteenth, `AUTHZ-3002`, is excluded
   from this criterion because §6.8.1.1 assigns it no producer**: it is a reserved identifier, nothing in
   this plan emits it, and a criterion demanding it could never be discharged by any correct
   implementation — 13 + 2 + 1 = 16, and the criterion is written against the 15. The
   criterion is discharged per producer as that producer comes into existence, not all at once: a gate
   that demanded all fifteen from the port would demand two classes from components the port does not
   contain, and it would be failed by a correct implementation.
   [03](03-modernization-roadmap.md) scopes its workstream gates accordingly.
2. **The field list enforced by test.** A redaction test over at least a checkout, a sign-in and an
   error path asserts that no record contains any never-logged value of 06 §9.2 — in particular no
   password, no personal-data field, no promo code, no session identifier, no cookie value, no
   anti-forgery token and **no raw login name** outside row 16.
3. **Collection verified rather than configured, and verified per producer.** Each class demonstrably
   arriving in the sink 06 §9.2 defines, at the retention 06 §9.5 sets. The application's **thirteen**
   classes are verified at the hosting workstream's exit, because that is where a real sink first exists.
   The **two** classes the command emits — `PROV-6001` and `AUTHZ-3001` — and
   `AUTHZ-3001`'s second producer, the Identity data migration, are verified at their own producers'
   exits, against the destination 06 §9.5.1 gives them — which is a **different** destination and reached
   by a different route: the four records leave the command on **standard output**, the release pipeline
   captures and selects them, and they are lodged as **one append-only blob** in the audit container
   06 §9.5 defines, never entering the application's telemetry path at all. So a single "all classes arrive in
   the sink" check would be asserting something untrue of those two classes, and the verification is
   **three** producer-and-class checks rather than two, because `AUTHZ-3001` has to be shown arriving from
   each of its two producers. **`AUTHZ-3002` is verified nowhere and is not a fourth check**: nothing
   emits it, so there is nothing to observe arriving, and a check that waited for it would never pass.
4. **The order-failure path proven to record something.** Today the checkout's bare `catch` discards the
   only evidence a write failed [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:58]; `ORDER-5002`
   is what replaces it, and a test that forces a persistence failure and asserts the record exists is
   what proves the replacement happened.
5. **The `SubjectPseudonym` key provisioned, scoped and proven to fail closed.** Its lifecycle table
   above is met in full: the secret exists per environment and per slot, resolves through the platform
   mechanism of 06 §8.4, is readable only by the runtime identity and the deployment principal, and is
   confirmed present by the deployment gate before traffic is admitted. **Four tests close it on this
   document's side**, and each one asserts an *event* property that the mechanism's own tests cannot reach.
   The grammar, canonicalization-stability, stale-label and environment-separation cases belong to
   [06 §8.4.7](06-azure-hosting-recommendations.md)'s four tests and are **not** duplicated here; where a
   test below overlaps one of them it names it and asserts the consequence rather than the parse:
   - **Positive — determinism and distinctness, as observed in emitted records.** Two failed sign-ins
     naming the same identifier produce records whose `SubjectPseudonym` values are **equal**, two
     different identifiers produce different ones, and — the case that matters most — a sign-in failure
     (`AUTH-1002`) and a duplicate-registration failure (`ACCT-2001`) naming **one** identifier produce the
     **same** value. That is requirement 1's single subject type asserted at both call sites, and it is
     exactly what 06 §8.4.7's algorithm-level stability test (its test 3) cannot show: that test exercises
     the function, and two classes can each call a correct function with a different subject type.
   - **Negative — fail closed, on the whole of the owner's validation and not a length check.** With the
     record absent, and with **each** of 06 §8.4.7's six startup checks failed in turn, the application
     refuses to start and **no** record is emitted carrying a raw or unkeyed identifier. Seven cases rather
     than one, because a single "malformed" case exercises whichever check the implementation happened to
     order first — and the environment-tag mismatch, check 3, is the one that must never be relaxed to a
     warning, so it is asserted on its own.
   - **Empty canonical result — the field is omitted and the record still exists.** Submissions that
     canonicalize to empty — whitespace only, and an empty value — assert an `AUTH-1002` record written
     **without** `SubjectPseudonym`, and specifically not carrying a constant label. That is requirement 2,
     and no grammar test can catch it, because the correct output is the **absence** of the field rather
     than a value of any shape.
   - **Malformed and stale-epoch labels — reported uninterpretable, and never counted as a subject.** The
     analysis path is driven with a label that does not parse, one whose scheme version is unknown, one
     whose environment tag is another environment's, and one whose key id resolves in no key of the current
     record — 06 §8.4.7's malformed case (its test 1) and stale case (its test 2), asserted here for their
     **security** consequence rather than their parse result. Each is asserted to be reported
     **uninterpretable and distinctly from "no match"**, and asserted **not** to contribute to any
     per-subject count this document reads off the field — `ConsecutiveFailureCount`, or the grouping by
     subject that makes the unthrottled guessing of F-09-03 and §3.3 visible at all. A parse failure that
     silently becomes a subject is a security conclusion drawn from a formatting error, which is
     requirement 3.
6. **Every outcome value of every class exercised at least once, and its actor and target asserted to hold
   exactly what the contract says — a value, a declared sentinel, or declared absence — and never an empty
   string, a null or a submitted value.** Not every class — every **outcome**: the table lists **45** outcome
   values across the sixteen classes, of which **41 are exercisable and 4 are reserved** — `AUTHZ-3002`'s
   single `Success`, which nothing emits, and the three values of row 16 reserved with it,
   `MembershipRevoked`, `MembershipNotHeld` and `Failed_UserNotFound`. **This criterion is written against
   the 41.** A reserved outcome has no invocation that reaches it, so a suite cannot exercise it and a
   criterion demanding that it do so would be unsatisfiable rather than strict; §6.8.1.1's note below the
   producer table states why the four are reserved and what would have to change for them to become
   testable. *The census moved by one when row 16 was reconciled to
   [05 §10.2](05-aspnet-core-migration-approach.md) property 4's four-record invariant: `NotAttempted` and
   `RunSucceeded` were added, both exercisable and both required by that invariant, and the overlapping
   `UserAlreadyExisted` was retired, so the class holds eighteen values of which fifteen are exercisable
   and the totals are 45 and 41 rather than 44 and 40. The four reserved values are unchanged.* The ones that break a naive implementation among the 41 are the failure
   outcomes where the entity does not exist. `ACCT-2001` at `DuplicateUserName` and at `ValidationFailed`,
   `ADMIN-4001` at `ValidationFailed`, `ORDER-5002` at `OrderId`-absent, and each of `PROV-6001`'s
   **five exercisable** failure outcomes — the six of row 16 less the reserved `Failed_UserNotFound` —
   must each produce a record asserted against that three-way rule, and on
   `Failed_ArgumentRejected` the target-username property is asserted **absent**, which is the rejection
   contract's rule 3 rather than an omission a suite may tolerate. `PROV-6001` has four outcomes a suite
   reaches only by choosing the run's shape, and they are named here so none is missed:
   `AlreadyPresent_NotRotated` on a provisioning run
   against an existing account that requested no rotation, `Rotated` on one that did, `RunSucceeded` on the
   run record of a run whose three operations all completed, and `NotAttempted` on the operation records of
   a run that stopped before reaching them — which the five rejection cases below already drive three at a
   time, so the two records to assert deliberately are the run record's success and an operation record
   left unreached by a **failure** rather than by a rejection. Its six failure
   outcomes are five for testing purposes, because `Failed_UserNotFound` is one of the four reserved
   values above. **And each
   failure record is asserted to carry no free-form
   text** — no `IdentityResult` description, no provider message, no exception message — which is the
   closed-outcome rule enforced rather than stated. A suite that exercises only the success outcome of
   each class passes while every failure path writes whatever its author had to hand.
   **`Failed_ArgumentRejected` is exercised under the rejection contract's own rule 6 rather than by a
   single case**: five rejections, one per `RejectedField` value, each asserting the **four-record**
   cardinality — three operation records at `NotAttempted` and the run record at the rejection, in
   [05 §10.2](05-aspnet-core-migration-approach.md) property 4's fixed order — that no store write and no
   privilege-change record accompanied them, that every field holds
   its declared sentinel or is declared absent on all four, and that the submitted value appears **nowhere in the
   serialized records** in any field or encoding. That last assertion is what makes this criterion a
   CWE-117 test rather than a completeness test.
7. **The three seams of §6.8.1.1 built, and each proven to change no behaviour.** Without them, three of
   the thirteen application-produced facts have no code path that can produce them, and the criteria above
   would be met by an implementation that silently omits them.
   - **Seam 1.** Seven cases: six behavioural ones, of which the third is the one that catches an audit
     record asserting a write that never happened, and a seventh that proves the object graph the other six
     run against can be constructed at all. **(a)** A sign-in by an account whose stored hash is in the legacy format asserts one
     `ACCT-2002` at `Success` carrying `RehashedOnSignIn = true`, alongside its `AUTH-1001`. **(b)** A
     sign-in by an account already at the current format asserts **no** `ACCT-2002`. **(c)** A sign-in by a
     legacy-hash account with the **store update forced to fail** — the `UpdateUserAsync` result made
     unsuccessful, by a concurrency-token conflict or an injected store fault — asserts **no** `ACCT-2002`
     **at all**, asserts that the distinct `credential-rehash` diagnostic **is** written at `Warning`
     carrying no error description and no hash, and asserts that the account's stored hash is **unchanged**,
     which is what proves the audit trail and the store agree. **(d)** A user-initiated password change on a
     legacy-hash account asserts **exactly one** `ACCT-2002`, the change-password one, **without**
     `RehashedOnSignIn` — never two records for one credential change. **(e)** **Two concurrent sign-ins
     for the same legacy-hash account** assert **exactly one** `ACCT-2002` with `RehashedOnSignIn` across
     both requests, the losing request's `credential-rehash` diagnostic, and **both** sign-ins succeeding —
     which is what proves the per-request scope of the observation and that emission follows each request's
     own `IdentityResult` rather than the hasher's advice. **(f)** A single sign-in by a legacy-hash account
     that **also** carries recorded access failures, so the request reaches `UpdateUserAsync` **twice** —
     the rehash write and then the access-failed reset — asserts **exactly one** `ACCT-2002`, which is the
     consume-once read proven rather than asserted. **(g)** **The registration shape itself, asserted rather
     than reviewed.** The application's service collection is built with **validate-on-build and
     validate-scopes enabled**, `IPasswordHasher<ApplicationUser>` is resolved **from a request scope**, and
     the resolved instance is asserted to be `RehashObservingPasswordHasher` holding the concrete
     `PasswordHasher<ApplicationUser>` as its inner hasher — after which case (e)'s two concurrent sign-ins
     are run against **that same provider**, so the graph proven resolvable is the graph that persists the
     rehash. This is a test rather than a review item because the self-referential shape the registration
     row rejects fails exactly **here**, at provider construction, and nowhere earlier: it compiles, it
     registers, and it is indistinguishable from the correct shape until something resolves it. Every
     sign-in in (a) to (c), (e), (f) and (g) succeeds, which is the no-behaviour-change half.
   - **Seam 2.** The **five** cases the seam-2 block enumerates, not one denial: the **challenge** path
     (unauthenticated) and the **forbid** path (authenticated non-administrator) asserted separately, each
     carrying `DenialKind`, `RequiredPolicy`, `ControllerName` and `ActionName`, and asserted to differ in
     exactly `DenialKind` and the actor; a **success** asserting **no** record; a **bare `[Authorize]`**
     denial asserting `RequiredPolicy = default`, which is the value the deployed application produces most
     often and the one an empty field would otherwise replace; and the **fallback** pair — the deployed
     `FallbackPolicy` asserted **null**, and `RequiredPolicy = fallback` asserted in a test-only host that
     registers one. Two paths rather than one because
     `PolicyAuthorizationResult` distinguishes them and the inner handler answers them differently, so a
     suite that exercises one proves nothing about the other. And in every denial case **the status code and
     `Location` header are asserted identical with the decorator registered and with the framework handler
     alone**, for a `GET` and for a non-`GET` request. That comparison is the only thing that proves the
     delegation is real rather than a reimplementation that happens to look similar.
   - **Seam 3.** The sequential and concurrent lockout tests of the seam-3 table. The concurrent one
     asserts **at least one** `AUTH-1003` and collapses every record for the account on exact equality of
     the durable de-duplication key `{UserId}|{LockoutEndUtc:o}` to **one logical event** — never an
     exactly-one-physical-record claim, which is a property this design does not have.
8. **All three externally supplied fields of the command's records treated as untrusted input, and the
   actor precedence proven rather than asserted.** The fourth convention of §6.8.1 is closed by test rather
   than by prose, in three groups.

   *Rejection, per field and per rule.* A `MUSICSTORE_AUDIT_ACTOR` value carrying a line break — or any
   other control
   character, or a character outside the closed set — is **rejected**, the command exits non-zero,
   **nothing in the store changed**, and the rejection's own run record names the failing field without
   reproducing the offending value; a `MUSICSTORE_AUDIT_ACTOR` value of 129 permitted characters is
   **rejected rather than
   truncated**; a **target username containing a character outside the user-name set
   [05 §6.1](05-aspnet-core-migration-approach.md) owns** — one of the five characters the framework default
   would admit and that policy does not is the discriminating input — is rejected **before any store call**,
   with the same three consequences;
   and a **role name outside the closed role set** is rejected the same way. Each of the four is a separate
   case, because a single filter over all three fields would pass the weakest of them.

   Two further cases complete the set, and both are ones a suite built from the four above would miss.
   **A supplied `MUSICSTORE_AUDIT_ACTOR` value that fails validation on a run where platform metadata *is*
   present** must be
   rejected too — the attested `Actor` is unaffected, so an implementation that validates only the value it
   is about to use as the `Actor` would let the failing claim through into `SuppliedActorClaim`; the
   rejection contract's `RejectedField = suppliedActorClaim` is exactly this case. And **a missing
   credential** on a provisioning run that needs one is `RejectedField = credential` at
   `RejectionRule = Missing`. With those two, the five `RejectedField` values of the rejection contract are
   each driven once, which is that contract's rule 6, and every one of the five asserts the four structural
   properties it fixes — **four-record** cardinality, three operation records at `NotAttempted` and the run
   record at the rejection, no store write and no privilege-change record, declared
   sentinels or declared absence in every field of all four, and **the submitted value nowhere in the serialized
   records**. The reserved **actor-side** literals are driven as inputs as well as asserted as outputs: a
   `MUSICSTORE_AUDIT_ACTOR` value of `unattributed` or `rejected` is **refused**, which is the check that
   makes those two
   reservations real rather than declared. **And one further invocation shape is driven, because it is the
   one the channel forbids**: an invocation that passes the actor **on the command line** is rejected by
   the closed pre-parse of [05 §10.2](05-aspnet-core-migration-approach.md) property 1b, before any host
   is built, with **no store write and no business mutation** — the assertion that keeps the actor on the
   channel that is absent from process listings and from shell and pipeline history. **"Nothing written"
   means the store and the business state, never the audit trail, and the difference is asserted rather
   than left to the reader:** this path still emits the **four** `PROV-6001` records the rejection
   contract's rule 1 fixes — three operation records at `NotAttempted` and the run record at the rejection
   — because [05 §10.2](05-aspnet-core-migration-approach.md) property 4 makes four records per invocation
   an invariant that holds on the pre-host path too, and that path is the one the command's dedicated
   serializer exists for, there being no logger to resolve before a host. A suite asserting an empty audit
   stream here would be asserting the opposite of the contract it is meant to close.
   **The user-name side asserts the opposite, and
   asserting it is
   what keeps the target field's typed absence honest:** a `--username` of `none` or of `rejected` is
   **accepted** — both are legal user names, and refusing them would make a source account bearing either
   unmanageable through the audited command — and a rejection record produced on any other failing field
   carries **no target-username property at all** rather than a value drawn from that domain. A suite that
   asserted the refusal instead would be asserting the withdrawn reservation.

   *The actor precedence, in both directions and in disagreement.* Three invocations, asserted on the
   parsed record rather than on prose. **(a) Platform metadata present and the supplied
   `MUSICSTORE_AUDIT_ACTOR` value disagreeing with
   it**: `Actor` equals the **platform** value, `SuppliedActorClaim` equals the **supplied** value,
   `ActorSource` is `platformMetadata`, and the run is **not** refused — the disagreement is recorded, not
   treated as an error, which is the property that makes the field worth having. **(b) Platform metadata
   present and agreeing**: the same three fields, with `Actor` equal to `SuppliedActorClaim` and
   `ActorSource` still `platformMetadata` — so the record distinguishes an attested value from a fallback
   even when the two strings match. **(c) Platform metadata absent**: `Actor` equals the validated
   `MUSICSTORE_AUDIT_ACTOR` value and `ActorSource` is `environmentVariable`. A test that only exercises
   (c) would pass an
   implementation with no precedence at all.

   *Record integrity.* Every record a successful run emits **parses as exactly one structured JSON object**
   whose `Actor`, `SuppliedActorClaim`, `ActorSource`, target-username and `RoleName` properties equal the
   accepted values — asserted by parsing the record and reading the fields, never by matching text in a
   rendered line, and asserted to be **one** object per record so that no accepted value can have forged a
   record boundary (CWE-117). This is what makes the rejection tests defence in depth rather than the only
   defence.

#### 6.8.2 Personal-data governance — the requirement that closes F-09-33

**F-09-33 records an absence in the source; this is the control set the target must have before personal
data is loaded into it.**

**The scope is THREE personal-data stores, not two, and they are named here individually so that no
consumer's gate can be scoped to a subset of them.** A reader who has to infer the third from a phrase
about a field will scope a workstream to the two obvious ones, which is precisely what happened once
already and is why the list is enumerated rather than described:

| # | Store | The personal data it holds | Who owns the store |
| --- | --- | --- | --- |
| **1** | **The catalog and order store** — the ported application's domain database | The **nine order personal-data fields** enumerated above, plus the `Username` link that ties an order to an account [src/MVC5/MvcMusicStore/Models/Order.cs:18] | The domain data migration; deliverable 05 for the schema, deliverable 06 for the instance |
| **2** | **The Identity and membership store** — the account records the `Username` link resolves to | Account records: user name, email, phone number where present, and the credential material beside them. Its shipped form is the Identity 1.0 schema in MVC 5 and the SimpleMembership and ASP.NET Membership stores in MVC 4 and MVC 3 (§3.1, §4.1, §5.1) | The Identity data migration; deliverable 05 |
| **3** | **The security-event store** — the durable audit trail §6.8.1's sixteen event classes are written to | The **`ClientIpAddress`** field that seven of those sixteen classes carry, alongside a pseudonymous but resolvable actor. It is personal data on any reading: an address plus a timestamp is attributable to a subscriber line | **The store itself, its transport and its retention are deliverable [06](06-azure-hosting-recommendations.md)'s** — 06 §9.5 for the store of record and retention, 06 §9.2 for the sink and the log-privacy policy. **What must be governed in it is this sub-section's** |

**Store 3 is the one a governance gate loses, and losing it would be a gap created by a security
control.** It is not a store the source application has; it is a store *this document's own audit
requirement introduces* (§6.8.1), and it holds a personal-data field no other store holds. An earlier
form of this scope governed the personal data the *application* stores and stopped there — two stores —
leaving the third ungoverned by construction. So the count is stated as **three** in the finding register
row for F-09-33, in this scope statement, and in each of the six controls below that touches more than one
store; a control that names only "the personal-data tables" is read against all three of the rows above,
and **a downstream workstream, gate or exit condition that covers two of the three does not satisfy this
requirement.**

Because the three stores have three different owners, **no single migration workstream owns the control
set.** Deliverable 03 owns it as an **approval-gated workstream** whose conditions gate both the rehearsal
against a real copy and the production load, and its scope is the three stores above rather than the two
migrated ones; deliverable 07 sizes it and carries the risk as its register entry R15; deliverable 06 owns
encryption at rest, backup retention, and the audit store and its retention. **This
sub-section states the requirement and what counts as meeting it, and nothing else — the periods
themselves are the data owner's to set and are deliberately not invented here.**

**Why the requirement is stated here rather than left to the migration.** The exposure changes even
though the absence is inherited. Today the nine fields sit in a file-attached local database with no
external reachability [src/MVC5/MvcMusicStore/Web.config:12-13]. After the port they sit in a managed
network-reachable database with scheduled backups and restore points that outlive the rows. Every part of
that is an improvement in availability, and every part widens the surface over which an ungoverned
personal-data set persists.

| # | Control | What counts as meeting it | Approver |
| --- | --- | --- | --- |
| 1 | **Retention period per data class** | A stated period and a stated basis for each of: order personal data, account records, security-event records and the provisioning audit trail. The last two are aligned to 06 §9.5 rather than set independently, so one retention decision does not silently override another | Data owner, with security |
| 2 | **Handling rules for non-production copies** | Whether a restored copy of either shipped store may exist outside production at all; if so, under which access restriction, for how long, and with what evidence of destruction when the rehearsal ends. This is the control that gates rehearsing the data migrations against real data | Data owner, with security |
| 3 | **Legal hold** | A recorded mechanism by which a hold is placed, honoured against the deletion operation, and released — so that a deletion obligation and a preservation obligation cannot both be satisfied by guesswork | Legal, with the data owner |
| 4 | **Deletion or field-level anonymization** | An implemented operation, demonstrated against **synthetic** personal data before any real record passes through it, run from the release path under the deployment principal and never from the application under its runtime identity — the same separation 06 §6.2 requires of DDL. It must handle the case F-09-33 names: an account deleted while its orders persist unreferenced | Data engineering, with security |
| 5 | **Deletion propagation into backups** | The maximum period for which deleted personal data remains recoverable from a backup or restore point, stated explicitly and reconciled against 06 §6.7's backup retention. An undefined window means a deletion that is not a deletion | Data owner, with platform engineering |
| 6 | **Actor-attributable read auditing over the personal-data tables** | **Every read of the nine fields is attributable to the principal that performed it, whichever path performed it — which makes this a requirement on the *database*, not on the application.** A record the application emits covers only the application's own read paths and is silent for every other reader the target admits: the deployment identity inside its change window, the two temporary migration identities (schema extraction and data load), the monitoring identity, the jobs identity, and any interactive query an operator runs. **The mechanism is [06](06-azure-hosting-recommendations.md)'s and is cited rather than asserted here** — the audit action set that captures reads of the PII-bearing tables, the dedicated immutable sink, the connecting principal as the recorded actor, the containment that stops the audit of who read personal data from itself becoming a further copy of it — and containment rather than redaction, because the platform's audit record carries the statement text and offers no supported filtering of the values inside it, so what bounds the exposure is the audited action set being scoped to reads of those tables rather than to every statement, plus the record's own classification and the audit store's restricted readership — the retention ([06 §9.5.3](06-azure-hosting-recommendations.md#pii-read-audit)), the reader coverage across the identity inventory 06 owns, and the test that a read by each named identity produces a record. **This document states the requirement; it defines no trail of its own and may not be read as evidence that one exists.** The residual risk is named rather than absorbed: a read that never traverses the audited connection — a restored copy, a backup file, or the non-production copy control 2 admits — produces no record, which is why controls 2 and 5 govern the copies and why the reader inventory has to stay exhaustive as identities are added. This is what closes the "no access log" half of F-09-33, and it stays closed only while the reader inventory and the audited action set agree | Security, with platform engineering |

**`ClientIpAddress` — how the six controls apply to it, because a field this document requires is a field
this document must govern.** Eight of the sixteen classes in §6.8.1 carry it — rows 1 to 7 and row 10 — and it is personal data on any
reading: an address plus a timestamp is attributable to a subscriber line, and in this catalog it sits
beside a pseudonymous but resolvable actor. No seventh control is added; each of the six is read onto this
field, and four points are settled here rather than left to the emitter.

- **Two uses, and only one of them is in scope.** The **transient** full address used to partition the rate
  limiter ([05](05-aspnet-core-migration-approach.md)) lives in process memory for the life of a request
  and is not persisted; the **persisted** value in an audit record is the governed one. Conflating them
  would either govern a variable or leave a stored field ungoverned.
- **The value is canonical and it is derived, never taken raw.** It is the client address as resolved by
  the forwarded-header trust configuration [06](06-azure-hosting-recommendations.md) owns — never a raw
  request header, which is caller-controlled and would put attacker-chosen text in an audit field. It is
  written in one canonical form so that two records for one client compare equal: an IPv6 address in its
  normalized textual form, an IPv4-mapped address written as IPv4, and **no port**, which identifies a
  connection rather than a client and would defeat every comparison the field exists for.
- **Full precision is governed rather than reduced, and the alternative is named as a decision rather than
  taken here.** The field is retained at full precision because its security purpose is correlating
  attempts across time ( §3.3, F-09-03), which a truncated address weakens exactly where the field is
  useful. **Reducing persisted precision — recording a network prefix instead of the address — is a
  legitimate data-owner decision with a real cost**, and it is left as one: this document invents no prefix
  length, because a threshold presented as a finding is a decision in disguise, which is the same rule this
  sub-section applies to retention periods.
- **The six controls, read onto the field — against the mechanism the owner now specifies, which is a log
  channel rather than a database object.** Every class in §6.8.1 that carries this field is
  application-produced, so the field exists only in the ingested log channel
  ([06 §9.5](06-azure-hosting-recommendations.md)) and never in the separate destination
  [06 §9.5.1](06-azure-hosting-recommendations.md) assigns the two tooling producers' records.
  **Control 1**: its retention is that channel's, and satisfaction now
  turns on a **workspace table setting rather than on a store's own lifecycle** — the interactive
  `retentionInDays` and the long-term `totalRetentionInDays` written and read back on the one table those
  records land in, which [06 §9.5.2](06-azure-hosting-recommendations.md) owns together with the readback
  that proves it — and it is never a longer or separate period, so there is no second copy with a lifetime
  of its own. Where the platform declines a table-level setting on that table, 06 §9.5.2 escalates it to
  this document's security owner as a scope gap rather than leaving the table on a workspace default, which
  is the behaviour this control requires of it. **Control 2**: a non-production copy of audit data is a
  copy of this field and is governed as one. **Control 3**: a hold cannot be placed on an ingested record, so
  control 3 is **not satisfied on this field** and is recorded as an accepted gap immediately below rather
  than asserted through a mechanism that no longer exists. **Control 4**: **the reconciliation this field
  forces is stated rather than glossed** — an ingested record is not individually mutable and the channel
  offers **no per-record deletion operation**, so control 4's deletion or anonymization operation does
  **not** apply to this field and must not be specified as though it did; what bounds the field is
  **retention expiry**, and a subject-deletion response therefore states that the security-event trail is
  retained for its stated period rather than implying a deletion the platform does not offer. The claim is
  exactly that and no more: 06 §9.5 records that the channel is **not** write-once, so a principal holding
  workspace management rights can delete records wholesale even though nothing can edit or remove one —
  [06 §6.6](06-azure-hosting-recommendations.md) keeps the application's runtime identity out of that set.
  **Control 5**: this field is in no database backup and no restore point, because it is not in the
  application database — so control 5's propagation window for it is not
  [06 §6.7](06-azure-hosting-recommendations.md)'s backup retention but the long-term tier of control 1,
  which expires with it, and that is the window a subject-deletion response states. **Control 6**: reads of
  it are attributable on the same terms as reads of the nine order fields, and a subject-access response
  covers it.

**One control cannot be satisfied on this field, and it is recorded as an accepted gap with an owner rather
than asserted.** Control 3 requires a mechanism by which a legal hold is placed on data, honoured against
the deletion operation, and released. For the order and account stores it is a database-level mechanism and
control 3 stands. For the security-event records — and therefore for this field —
[06 §9.5](06-azure-hosting-recommendations.md) **withdrew** the immutable store that would have carried a
locked retention window and a hold, on the ground that it is infrastructure outside the AAP artifact set of
AAP §0.3.1, §0.4.1 and §0.2.2; the ingested channel that replaced it has no hold primitive. **The
consequence, stated plainly: no preservation obligation can be honoured against a security-event record
beyond its retention expiry, and none can be shortened before it either.** What stands in its place is
partial and is named as partial — retention expiry enforced by the per-table setting of
[06 §9.5.2](06-azure-hosting-recommendations.md), and tamper-*evidence* through the record integrity field
[06 §9.5](06-azure-hosting-recommendations.md) defines, which detects alteration and does not prevent
deletion. **Closing it needs artifacts the AAP set does not contain**, so it is not a defect either
document can remedy inside this scope: the owners are **this document's security owner jointly with the
approver of the AAP artifact set**, which is the same escalation 06 §9.5 records from the hosting side, and
the decision is theirs to take when the artifact set is next opened. Until then the gap is carried, not
closed — and a subject-deletion or preservation response that implied otherwise would be asserting a
property the platform does not provide.

**Two things this requirement deliberately does not do.** It does not set a retention period, because
that is a decision with a named approver and inventing one here would disguise a decision as a finding.
And it does not extend to payment data, because §3.11 establishes there is none: no card, account or
payment-token field exists on any entity in any edition, which is the single largest bound on this
finding's severity and is restated here so the control set is not over-scoped.

**The gate, and the scope the gate is read against.** Deliverable 03 makes controls 1–3 an entry
condition of the two data-migration workstreams and all six an entry condition of cutover. **Each of
those conditions is scoped to all three stores of the scope table above — the catalog and order store,
the Identity and membership store, and the security-event store — and a gate satisfied for two of the
three is not satisfied.** The two data-migration workstreams touch stores 1 and 2 only, which is exactly
why the cutover condition rather than the migration conditions is where store 3 is caught: the
security-event store first holds a `ClientIpAddress` when the ported application first runs, not when a
migration loads it. The operative consequence is one sentence: **no production
personal data is loaded, and no cutover completes, until all six controls are in place over all three
stores.** A migration that lands nine personal-data fields
into a hosted database with no retention period and no deletion path is a governance failure created by
the migration, not inherited from the source — and it is the one failure in this document that is
cheapest to prevent and most expensive to remedy after the fact.

**The gate is unaffected by the accepted gap above, and saying so is what keeps the two readable together.**
The gate is on the six controls **over the order and account stores**, where all six including control 3
have a mechanism; the gap is control 3's application to the one personal-data field the audit requirement
itself introduces, and it is carried by the owners named above rather than blocking the load. A gate that
treated the audit-side gap as unmet would stop a migration for a scope decision no migration can take.

### 6.9 The credential-shaped databases themselves are committed to source control

**Editions: MVC 3, MVC 4 and MVC 5.**

**Finding F-09-34 — a credential-shaped database is tracked in git in each of the three edition
trees: MVC 4's and MVC 5's own credential stores with their password material intact, and in MVC 3's
tree a tutorial membership database that no tracked configuration references.** Severity **Critical**.

Fourteen database binaries totalling exactly **43,376,640 bytes** are tracked (§9). Three of the
seven `.mdf` files carry a membership or identity schema — one in each edition tree. The fourth
column is the distinction that matters and is stated per row rather than assumed:

| Edition tree | Committed credential-shaped database | Schema | That edition's runtime store? |
| --- | --- | --- | --- |
| MVC 3 | `src/MVC3/MvcMusicStore-Assets/Data/ASPNETDB.MDF` (10,485,760 bytes) with `aspnetdb_log.ldf` | classic ASP.NET Membership | **Not assigned by this repository** — a tutorial asset [src/MVC3/readme.txt:11-12], referenced by no tracked configuration (`git grep -il 'ASPNETDB' -- 'src/'` → 0; §9). The edition's effective store is undetermined — F-09-16, §5.5 |
| MVC 4 | `src/MVC4/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20120831200627.mdf` with its log | SimpleMembership | Yes — reached through `DefaultConnection` [src/MVC4/MvcMusicStore/Web.config:12-17] and created by the application itself [src/MVC4/MvcMusicStore/Filters/InitializeSimpleMembershipAttribute.cs:37] |
| MVC 5 | `src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205.mdf` with its log | ASP.NET Identity 1.0 | Yes — bound to `DefaultConnection` by the Identity context [src/MVC5/MvcMusicStore/Models/IdentityModels.cs:13] against [src/MVC5/MvcMusicStore/Web.config:12] |

A byte-wise UTF-16LE name probe of each finds membership tables and password columns present (§9
carries the probe, and the locator for a binary is the evidence form and the artifact's size — §1.4,
so each citation below carries both the offsets the probe reports and the size of the file they sit
inside). Every offset below is the **first occurrence at any byte alignment**, which is the
distinction the probe in §9 is built around: three of these names occur first at an *odd* byte offset,
so a probe that decoded the file once from offset 0 would report a later occurrence and call it the
first. MVC 3's contains
`aspnet_Membership`, `aspnet_Users`, `aspnet_Roles`, `Password` and `PasswordSalt`
[src/MVC3/MvcMusicStore-Assets/Data/ASPNETDB.MDF:first UTF-16LE byte offsets at any alignment 0x1C4AE, 0xE8F62, 0x1C700, 0x5485E, 0x548F4 respectively, in a 10,485,760-byte file];
MVC 4's contains `webpages_Membership`, `webpages_Roles`,
`UserProfile`, `Password` and the literal `Administrator`
[src/MVC4/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20120831200627.mdf:first UTF-16LE byte offsets at any alignment 0xEDAD0, 0xEDBE2, 0xED9DA, 0x73823, 0x3AAA0 respectively, in a 3,211,264-byte file];
MVC 5's contains `AspNetUsers`,
`AspNetRoles`, `PasswordHash`, `SecurityStamp` and the literal `Administrator`
[src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205.mdf:first UTF-16LE byte offsets at any alignment 0xECBF2, 0xECB9C, 0x732E9, 0x73336, 0x3700B respectively, in a 3,211,264-byte file].
**A name probe is evidence, not proof** —
it cannot distinguish an absent column from one stored in a form the probe does not surface, and the
alignment correction above changes only *where* a name was found, never *whether* it is there.

What follows from it, bounded per edition rather than generalized:

- **For MVC 4 and MVC 5 the conclusion is not in doubt: the password hash of the seeded
  administrator account of F-09-05 is in this repository and in its history.** Each of those two
  editions provisions that account at startup from the committed credential
  [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:23-24],
  [src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:23-24], MVC 5's own documentation states the seeded
  account as a fact [src/MVC5/README.md:75-85], and the literal `Administrator` is present in both
  committed stores at the offsets above.
- **For MVC 3 no account material is inferred at all.** The edition provisions nothing (F-09-18), the
  file is a tutorial asset no tracked configuration reads (§5.5), and its effective runtime store is
  undetermined (F-09-16). What is established is only what the probe shows: a committed database
  carrying a membership schema with `Password` and `PasswordSalt` columns. That is a real exposure —
  credential-shaped data in source control — and it is why this finding covers all three trees; it is
  not evidence about any deployed MVC 3 account.

Three aggravating facts:

1. **They are tracked despite being excluded.** `.gitignore` lists `App_Data/` at [.gitignore:32],
   which matches each of these files that sits under an `App_Data/` folder — the pattern's case is
   those directories' own, so this match does not depend on the host — and an ignore rule cannot
   untrack a file already added. So this is not a deliberate decision anyone made — it is a mistake
   nobody noticed, which is why deliverable 08 records it as debt rather than as design. The committed
   `packages/` payloads in the same gitignored-yet-tracked set are held by a weaker rule, and the
   distinction matters where §9.1 uses it: `Packages/` at [.gitignore:33] is the only rule that
   matches them, and it matches only because this host reports `core.ignorecase = true` — on a
   case-sensitive checkout no rule matches them at all — while `packages/*` at [.gitignore:15] is
   anchored by its interior separator to the directory holding the `.gitignore` and so matches nothing
   nested. [04 — .NET 8 migration strategy](04-dotnet8-migration-strategy.md) §A.6 owns that
   analysis and it is cross-referenced rather than restated. MVC 3's file is the one exception to the
   mechanism rather than to the finding: it sits under `src/MVC3/MvcMusicStore-Assets/Data/` rather
   than any `App_Data/` folder, so no ignore rule ever matched it —
   [11 — Cloud readiness assessment](11-cloud-readiness-assessment.md) records the resulting
   ten-and-four split of the same fourteen binaries and owns it.
2. **MVC 5's is also the only schema evidence for the Identity migration**, which is why it cannot
   simply be deleted without a plan — deliverable 05 depends on extracting the real schema from it.
   The security remediation and the migration requirement are in tension, and that tension belongs to
   deliverable 03's sequencing.
3. **Removal requires history rewriting or explicit acceptance.** Deleting the files from the working
   tree leaves every blob in the object database. Deliverable 08 owns that choice.

### 6.10 `customErrors` never appears as a live element

**Editions: MVC 3, MVC 4 and MVC 5.**

**Finding F-09-35 — error-detail behaviour is left entirely to a framework default that no edition
states, while every edition ships `debug="true"`.** Severity **Medium**.

**The finding is the liveness, not the count** — no loaded configuration file in any edition declares
`<customErrors>`, so production error-detail behaviour is whatever the framework defaults to. The
counts are stated below only because they are easy to conflate, and each one is given with its unit
and the command that produces it:

| Unit counted | Value | Command |
| --- | --- | --- |
| Occurrences of the **word** `customErrors` in the six XDT transform files | **24** (four per file) | `git ls-files -- '*Web.Debug.config' '*Web.Release.config' \| grep -v '/packages/' \| xargs grep -o 'customErrors' \| wc -l` |
| Occurrences of the **literal** `<customErrors` in the same six files | **12** (two per file) | `git ls-files -- '*Web.Debug.config' '*Web.Release.config' \| grep -v '/packages/' \| xargs grep -o '<customErrors' \| wc -l` |
| Of those twelve, **element opening tags** of the commented example | **6** (one per file) | e.g. [src/MVC5/MvcMusicStore/Web.Release.config:25]; the other six are a prose mention of the section name inside the same comment, e.g. [src/MVC5/MvcMusicStore/Web.Release.config:21] |
| **Live** `<customErrors>` elements, in any edition | **0** | `git grep -n 'customErrors' -- 'src/MVC5/MvcMusicStore/Web.config' 'src/MVC4/MvcMusicStore/Web.config' 'src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config' \| wc -l` → `0` |

Both counting commands were run against this checkout and returned 24 and 12 respectively; the third
returned 0. **A word match is not an element**, which is why the units are named: the twenty-four
figure counts a name, not a configuration, and quoting it as "24 elements" would overstate what the
repository contains by a factor of four.

The liveness claim rests on separate evidence rather than on the counts: every occurrence lies between
an XML comment open and close, as the representative block
[src/MVC5/MvcMusicStore/Web.Release.config:19-29] shows — the comment opens at `:19`, the example
element runs `:25-28`, and the comment closes at `:29` — and the equivalent block in each of the other
five files has the same shape. Nothing in that range is applied by an XDT transform or read by the
runtime.

With `<customErrors>` undeclared, ASP.NET's default is `RemoteOnly`, which has two consequences worth
separating:

- **`HandleErrorAttribute` does not engage for local requests.** The filter each edition registers —
  [src/MVC5/MvcMusicStore/App_Start/FilterConfig.cs:10],
  [src/MVC4/MvcMusicStore/App_Start/FilterConfig.cs:10],
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Global.asax.cs:17] — acts only when custom errors
  are enabled for the request. A request originating on the server host itself therefore receives the
  detailed ASP.NET error page, complete with exception type, message and stack trace, rather than the
  generic `Error.cshtml`. On a single-tier host this is a narrow exposure; behind a reverse proxy or a
  platform front end that presents requests as local, it is not narrow at all, and no edition asserts
  otherwise.
- **`debug="true"` is what the committed configuration carries** —
  [src/MVC5/MvcMusicStore/Web.config:33], [src/MVC4/MvcMusicStore/Web.config:34],
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:16]. Each edition's Release transform
  removes the attribute, and that single transform is the **only** active XDT operation in any of the
  six files — [src/MVC5/MvcMusicStore/Web.Release.config:18] — so a Release publish does fix it.
  Two things it does not fix: the Debug transform is entirely commented
  [src/MVC5/MvcMusicStore/Web.Debug.config:17-29], and **no transform sets `customErrors` at all**,
  the example that would have done so being commented out immediately below the one active line
  [src/MVC5/MvcMusicStore/Web.Release.config:19-29]. So a Release build turns debug compilation off
  and still never states the error-detail policy.

The finding is therefore not "errors leak" — §3.10, §4.9 and §5.9 establish that no edition's error
*view* discloses anything. It is that error-detail policy is undeclared, so what a deployed instance
shows depends on a framework default and on where the request appears to come from.

### 6.11 Scaffolded-but-disabled external login ships as deployed attack surface

**Editions: MVC 4 and MVC 5** (MVC 3 has no external-login surface at all — §5.10).

**Finding F-09-36 — eleven external-authentication packages are deployed across two editions to serve
zero enabled providers.** Severity **Medium**.

**Two units, both counted, because one of them is easy to reach for and wrong.** The finding's unit is
**every package in the two editions' manifests whose only purpose is external authentication**, and there
are **eleven** of them — seven in MVC 4 and four in MVC 5. A narrower unit is also useful and is stated
here so nobody conflates the two: packages implementing a **named identity provider** number **four**, all
of them MVC 5's. Where a figure appears below, it says which unit it is counting.

| # | Package | Edition | Locator | Unit |
| --- | --- | --- | --- | --- |
| 1 | `Microsoft.Owin.Security.Facebook` | MVC 5 | [src/MVC5/MvcMusicStore/packages.config:20] | Named provider |
| 2 | `Microsoft.Owin.Security.Google` | MVC 5 | [src/MVC5/MvcMusicStore/packages.config:21] | Named provider |
| 3 | `Microsoft.Owin.Security.MicrosoftAccount` | MVC 5 | [src/MVC5/MvcMusicStore/packages.config:22] | Named provider |
| 4 | `Microsoft.Owin.Security.Twitter` | MVC 5 | [src/MVC5/MvcMusicStore/packages.config:24] | Named provider |
| 5 | `DotNetOpenAuth.AspNet` | MVC 4 | [src/MVC4/MvcMusicStore/packages.config:3] | Protocol stack |
| 6 | `DotNetOpenAuth.Core` | MVC 4 | [src/MVC4/MvcMusicStore/packages.config:4] | Protocol stack |
| 7 | `DotNetOpenAuth.OAuth.Consumer` | MVC 4 | [src/MVC4/MvcMusicStore/packages.config:5] | Protocol stack |
| 8 | `DotNetOpenAuth.OAuth.Core` | MVC 4 | [src/MVC4/MvcMusicStore/packages.config:6] | Protocol stack |
| 9 | `DotNetOpenAuth.OpenId.Core` | MVC 4 | [src/MVC4/MvcMusicStore/packages.config:7] | Protocol stack |
| 10 | `DotNetOpenAuth.OpenId.RelyingParty` | MVC 4 | [src/MVC4/MvcMusicStore/packages.config:8] | Protocol stack |
| 11 | `Microsoft.AspNet.WebPages.OAuth` | MVC 4 | [src/MVC4/MvcMusicStore/packages.config:23] | **The external-login API itself** |

4 + 6 + 1 = **11**, and 7 + 4 = **11** by edition.

**Row 11 was omitted from an earlier form of this finding, which counted ten, and it is the one row whose
omission mattered most.** `Microsoft.AspNet.WebPages.OAuth` is not a peripheral package: it ships the
`Microsoft.Web.WebPages.OAuth` namespace and the `OAuthWebSecurity` type that **is** MVC 4's entire
external-login API. MVC 4's authentication configuration imports that namespace at
[src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs:5] and every commented-out registration in the file calls
that type [src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs:17-29], and its account controller calls it
throughout [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:228-330]. So a count of ten described
MVC 4's protocol libraries while leaving out the assembly that hosts the code path the second risk below is
about — the reverse of the priority the finding claims.

`Microsoft.Owin.Security.OAuth` [src/MVC5/MvcMusicStore/packages.config:23] is deliberately **not** in the
table. It is OAuth server and bearer-token infrastructure rather than an external-authentication package,
which is exactly why MVC 5 ships five `Microsoft.Owin.Security.*` entries beyond cookies while only four of
them are providers — the discrepancy that makes the narrow unit worth naming.

**Every provider registration is commented out, and that — not a missing credential — is why the surface
is inactive.** The distinction matters because the prerequisites are **not uniform across the four
providers**, so a single sentence about client ids and secrets is wrong about one of them. In MVC 5 the
four registrations occupy [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:22-35]:
`UseMicrosoftAccountAuthentication` with `clientId` and `clientSecret`
[src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:23-25], `UseTwitterAuthentication` with `consumerKey`
and `consumerSecret` [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:27-29] and
`UseFacebookAuthentication` with `appId` and `appSecret`
[src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:31-33] are each written with **empty string
arguments** — while `app.UseGoogleAuthentication();`
[src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:35] **takes no arguments at all**. MVC 4's four are the
same shape at [src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs:17-29]: `RegisterMicrosoftClient`
[src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs:17-19], `RegisterTwitterClient`
[src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs:21-23] and `RegisterFacebookClient`
[src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs:25-27] with empty credential pairs, and
`OAuthWebSecurity.RegisterGoogleClient();`
[src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs:29] with none.

So the per-provider prerequisite is: **three of the four require a credential pair that does not exist
anywhere in the repository, and the fourth requires only that the comment markers be removed — its call
is complete as written.** That is a repository fact and it is the one that changes the assessment: the
single line standing between this application and a live external-provider registration is a `//`, not an
absent secret. It is *not* a claim that the provider would successfully authenticate anyone today —
whether the remote endpoint that credential-free call targeted still answers is a live-service question,
and this document is not a penetration test (§1.2) — and the reachability mark for the capability is
deliverable 01's [01 §9.3](01-architecture-overview.md).

*An earlier form of this paragraph read "Microsoft Account, Twitter, Facebook and Google, each with empty
credential arguments", which was true of three of the four and wrong about Google, and it attributed the
surface's inactivity to the empty credentials rather than to the comment markers. Both halves are
corrected above.*

Yet the packages ship: deliverable 02 records
four dormant `Microsoft.Owin.Security.*` provider packages in MVC 5 (F-02-03) and six DotNetOpenAuth
packages in MVC 4 (F-02-08), all deployed to the output directory regardless — and the eleventh, MVC 4's
`Microsoft.AspNet.WebPages.OAuth`, ships with them.

Two distinct risks, and neither is "an attacker can sign in through Facebook":

- **Unused code is deployed code.** Eleven packages of authentication, protocol-parsing and account-linking
  logic — the broad unit — are present in the deployed applications, pinned at 2012–2013 versions that
  deliverable 02 §8.1 places in its aged-dependency class. They are attack surface in the loaded-assembly
  sense, and they are surface nobody is maintaining because nobody believes the feature is on. Package count
  is not assembly count and this document does not conflate them: several of these packages carry more than
  one assembly, so eleven is the floor on what is deployed, not the figure.
- **The application code path for external login is live even though no provider is.** MVC 5's
  external-login actions are routable — [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:200],
  [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:209], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:237], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:245], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:265] — and MVC 4's likewise
  [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:228-330]. With no provider registered the
  challenge cannot complete, so this is dead rather than dangerous today; the risk is that enabling
  one provider activates an entire untested account-linking and account-creation path — and for one of
  the four, **"enabling" is deleting two characters**, because the Google call in both editions needs no
  credential and is complete as written (see above), so the barrier is a comment marker rather than a
  secret somebody would have to obtain. The path it would activate includes the
  `ChallengeResult` type MVC 5 hand-rolls
  [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:394] and the callback that does **not**
  pass the anti-forgery key (§3.4).

Deliverable 05 owns the decision to re-establish or remove the surface during the port; deliverable 02
owns the package dispositions. This document records that the surface is deployed and disabled, which
is the worst of both states for an assessment: it costs everything and protects nothing.

---

## 7. Controls that are present

An assessment that lists only failures cannot be calibrated, and several findings above are only
legible against the places this codebase gets the same class of problem right. Each row below is a
control that exists, with its evidence and the editions it holds in. None of them is a mitigation for
a finding in section 8 unless the row says so.

| Control | Editions | Evidence |
| --- | --- | --- |
| Every account POST validates its anti-forgery token | MVC 5 (8 of 8), MVC 4 (7 of 7) | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:55], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:88], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:113], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:147], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:199], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:236], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:264], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:301]; [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:35], [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:67], [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:89], [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:119], [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:163], [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:227], [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:271] |
| Sign-out is a token-protected `POST`, posted from a real form | MVC 5, MVC 4 | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:300-301], [src/MVC5/MvcMusicStore/Views/Shared/_LoginPartial.cshtml:4-6]; [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:66-67], [src/MVC4/MvcMusicStore/Views/Shared/_LoginPartial.cshtml:5] |
| The account controller is deny-by-default, with per-action opt-out | MVC 5, MVC 4 | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:15]; [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:16] |
| The sign-in failure **message** does not distinguish an unknown user from a wrong password — a content control only, credited no wider: in MVC 5 the two branches are distinguishable by response *time* (**F-09-38**, §3.4), and in MVC 4 and MVC 3 the timing question is unexercised rather than answered | all three, message-level | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:68]; [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:47]; [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:58] |
| The post-sign-in redirect is guarded against open redirection | all three; MVC 3's is the strictest | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:382-392] via `Url.IsLocalUrl` at [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:384]; [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:46-47] adds four further checks |
| The order confirmation page verifies the signed-in user owns the order | all three | [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:71-73]; [src/MVC4/MvcMusicStore/Controllers/CheckoutController.cs:71-73]; MVC 3's equivalent in its own `Complete` action |
| An account-management action verifies the caller owns the resource it names — the only resource-level authorization check on an account operation in the repository | **MVC 4 only** | `Disassociate` compares the owner resolved from the submitted provider pair against the caller [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:126], with the resolution at [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:122] and the guarded deletion at [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:134]. MVC 5 needs no equivalent — its action binds the caller's own id into the call [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:117]; MVC 3 has no such surface (§5.10). This is MVC 4's sixth enforcement point (§4.2) |
| Cart removal is scoped to the caller's own cart, not just the row id | all three | [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:64-66]; [src/MVC4/MvcMusicStore/Models/ShoppingCart.cs:64-66]; [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:63-65]. Partially undone by F-09-26, which is a *read* on the same endpoint |
| The checkout bind list restricts model binding to the nine customer fields | MVC 5, MVC 4 | [src/MVC5/MvcMusicStore/Models/Order.cs:8]; [src/MVC4/MvcMusicStore/Models/Order.cs:8]. This is the control MVC 3 lacks — F-09-22 |
| `Username` and `OrderDate` are additionally set server-side after binding | all three | [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:40-41]; [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/CheckoutController.cs:40-41] |
| Three of the four administration `Find` results are null-checked and return 404 | **MVC 5, MVC 4 only** | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:33-35], [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:74-76], [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:106-108], and MVC 4 at the identical line numbers [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:33-35], [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:74-76], [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:106-108]. The fourth is the Low finding in §6.3. **MVC 3 does not have this control at all:** its four `Albums.Find` calls [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:31], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:68], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:96], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:106] are every one unguarded, and the edition contains no `HttpNotFound()` call anywhere — see [08 §5.4](08-technical-debt-register.md) for the per-edition distribution |
| Razor views are never served as content — every path and verb maps to the not-found handler | MVC 5 | [src/MVC5/MvcMusicStore/Views/Web.config:31-32] |
| No connection string stores a database password | all three | [src/MVC5/MvcMusicStore/Web.config:12-13]; [src/MVC4/MvcMusicStore/Web.config:15], [src/MVC4/MvcMusicStore/Web.config:21]; [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:55-59] |
| No error view in any edition renders an exception, a message or a stack trace | all three | MVC 5's is 9 lines and reads no model property [src/MVC5/MvcMusicStore/Views/Shared/Error.cshtml:7-8]; MVC 4's is 11 lines of generic apology; MVC 3's is 9 lines of the same |
| No raw-output helper is used anywhere — no `Html.Raw`, no `MvcHtmlString.Create` | all three | `git grep -nE 'Html\.Raw\|MvcHtmlString\.Create' -- 'src/**/*.cshtml' 'src/**/*.cs'` → `0` (§9) |
| No raw SQL is executed anywhere, so there is no SQL-injection surface | all three | `git grep -nE 'ExecuteSqlCommand\|SqlQuery\|SqlCommand\|CommandText' -- 'src/**/*.cs'` → `0` (§9). All data access is EF LINQ |
| `HttpContext.Current` is never used, so no ambient-context authorization mistake is possible | all three | `git grep -n 'HttpContext.Current' -- 'src/**/*.cs' 'src/**/*.cshtml'` → `0` (§9) |
| No payment data is collected or stored, in any edition | all three | `Order` has no card, account or payment-token property [src/MVC5/MvcMusicStore/Models/Order.cs:11-66] |
| Provisioning is idempotent per operation | MVC 4 only | [src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:29], [src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:32], [src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:35] — and MVC 5's failure to do this is F-09-07 |
| Provider exceptions are mapped to safe messages rather than surfaced | MVC 4 (one of two sites), MVC 3 | [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:105], [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:107]; [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:140-143], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:151]. The MVC 4 site that does not is F-09-14 |
| The cart is migrated before the authentication cookie is issued | MVC 3 only | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:43], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:45] — the inverse of MVC 5's F-09-04 |
| The echoed album title is HTML-encoded before entering the response | MVC 3 only | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/ShoppingCartController.cs:68] — the only output-encoding call in the repository, and F-09-23 records its loss |
| The AJAX response is written to the DOM with `.text()`, which does not interpret markup | all three | [src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:28] — which is why F-09-23 is Low rather than a scripting finding |

---

## 8. Finding register

Thirty-eight findings, **`F-09-01` to `F-09-38`, contiguous, with no gap and no reused identifier**.
Severity is defined in §1.6 and assigned on exposure in a hosted, internet-facing deployment. The
**Editions** column is the acceptance criterion for this deliverable: no row is edition-ambiguous, and
no finding is asserted for an edition whose evidence is not cited in the section named.

**This table is canonical and §8.1's two rollups are counts over it, not summaries maintained beside
it.** That is stated because an earlier form of §8.1 totalled **37** against a 38-row register: it had
dropped F-09-38 from the severity rollup entirely, and it had placed F-09-08 — which the register
states for **MVC 5, MVC 4** — in the *MVC 5 only* bucket while leaving it out of the pair bucket. Two
derived tables disagreed with the rows they were derived from, and a reader had no way to tell which of
the two was wrong. Three properties now make that class of drift visible rather than latent, and each
is checkable from this page:

- **Every row appears in exactly one severity bucket and exactly one edition bucket, and no row is
  excluded from either rollup for any reason** — not for lacking a consumer, not for being
  comparative, not for being the lowest severity in the set. Where a row fits neither a single-edition
  nor an all-editions bucket it gets a bucket of its own, which is why §8.1's edition rollup has seven
  rows rather than four. F-09-12 (no consumer) and F-09-38 (added last) are the two rows a hand-kept
  rollup dropped, and they are counted here like every other row.
- **The severity vocabulary is §1.6's four values and no fifth is introduced.** §1.6 defines Critical,
  High, Medium and Low; every row carries one of those four; §8.1 counts those four. An earlier form of
  §8.1 carried an **Informational** bucket holding F-09-12 — a severity §1.6 does not define and the
  register does not assign — so that row was simultaneously **Low** in the register and absent from the
  Low count. The bucket is withdrawn; §8.1 states why **Low** is the right severity for a comparative
  observation and does not need a fifth level to say it.
- **The Editions cell is one of exactly seven canonical strings**, so the edition rollup is an
  exact-match grouping rather than a reading of prose: `all three`, `MVC 3 only`, `MVC 4 only`,
  `MVC 5 only`, `MVC 5, MVC 4`, `MVC 4, MVC 3`, `MVC 5 vs MVC 4`. Eight cells previously used a
  near-miss spelling — `MVC 5` for `MVC 5 only`, `MVC 4` for `MVC 4 only`, and `MVC 4, MVC 5` for
  `MVC 5, MVC 4` — which is precisely how a bucket comes to be kept by hand and then to drift from its
  source. The script that regenerates both rollups, the identifier-contiguity check and the
  consumer inverse index of §8.3 from this table is in §9.4.

| # | Finding | Severity | Editions | Section | Consequence owned by |
| --- | --- | --- | --- | --- | --- |
| F-09-01 | Authorization is a handful of enforcement points with no defence in depth behind any of them — **five in MVC 5** (3 controller-level attributes + 1 in-action ownership check + 1 data-scope predicate) and **six in MVC 4** (the same five plus `Disassociate`'s owner comparison [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:126]). The counting rule is §3.2's; MVC 3's own six, differently composed, is §5.2's | Low | MVC 5, MVC 4 | §3.2, §4.2 | 05 |
| F-09-02 | The entire authentication policy — password, lockout, confirmation, cookie lifetime, `Secure`, `SameSite` — is inherited from framework defaults and stated nowhere | Medium | MVC 5, MVC 4 | §3.3, §4.3 | 05 |
| F-09-03 | There is no account lockout in the sign-in code path, and Identity 1.0 predates the feature; the shipped store's physical columns are extraction-gated (§3.3). Guessing is unthrottled and, per F-09-32, unrecorded | High | MVC 5 only | §3.3 | 05 |
| F-09-04 | Sign-in issues the authentication cookie before the cart write it triggers, across two contexts, with no transaction and no logging | Medium | MVC 5 only | §3.4 | 05 |
| F-09-05 | A working administrator username and password are committed in cleartext and republished in the README | **Critical** | MVC 5, MVC 4 | §3.5, §4.5 | 05 |
| F-09-06 | Administrator provisioning runs as `async void`, so a provisioning failure is silent | Medium | MVC 5 only | §3.6 | 05 |
| F-09-07 | Provisioning is idempotent overall but not per operation; a partial prior run is never repaired | Medium | MVC 5 only | §3.6, §4.6 | 05 |
| F-09-08 | Five state-changing POST actions do not validate the anti-forgery token in **each** newer edition — 5 of 13 in MVC 5, 5 of 12 in MVC 4 — and they are the same five surfaces at the same line numbers; in MVC 5 two of the five emit a token that nothing checks | High | MVC 5, MVC 4 | §3.7, §4.7 | 05 |
| F-09-09 | The runtime identity must hold schema-modifying rights it needs only on first run | Medium | MVC 5 only | §3.9 | 06 |
| F-09-10 | The checkout wraps its entire write path in a bare `catch`, discarding the exception and showing no message | Medium | all three | §3.10, §6.8 | 05 |
| F-09-11 | The Forms authentication ticket lifetime is 48 hours, unjustified and unrevocable | Low | MVC 4 and MVC 3 | §4.1, §5.3 | 05 |
| F-09-12 | Broader token *emission* in MVC 5 makes a view-level anti-forgery audit actively misleading; emission is not coverage | Low | MVC 5 vs MVC 4 | §4.7 | — |
| F-09-13 | MVC 4's runtime identity must be able to create databases and tables, and the application does both at runtime | High | **MVC 4 only** | §4.8 | 06 |
| F-09-14 | Raw-exception disclosure **in the controller, not the error view**: `catch (Exception e) { ModelState.AddModelError("", e); }` [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:211-213] puts the exception object on the channel the validation summary renders to the user — latent under the shipped default helper, actual under any rendering that reads the error | Medium | **MVC 4 only** | §4.9 | 05 |
| F-09-15 | The same line also destroys the exception for the operator: not shown, not rethrown, not logged | Medium | **MVC 4 only** | §4.9 | 05 |
| F-09-16 | No credential store is declared, so password, lockout and storage policy are properties of the host and cannot be assessed from the repository | High | **MVC 3 only** | §5.1, §5.3 | 10 (host verification) |
| F-09-17 | Every account the edition registers is created with the same two hard-coded password-recovery literals — one fixed question and one fixed answer, both compile-time string literals occupying the fourth and fifth argument positions of the registration call [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:94]. **Both values are withheld under §1.3**; the exposure is conditional on the host provider's own default posture | High | **MVC 3 only** | §5.1 | 10, 05 |
| F-09-18 | The administration surface is guarded by an `Administrator` role no code creates, making it unreachable as shipped and gated on an external store | High | **MVC 3 only** | §5.2 | 01, 07 |
| F-09-19 | The account controller is allow-by-default, where both newer editions are deny-by-default | Medium | **MVC 3 only** | §5.2 | 07 |
| F-09-20 | Sign-out is reachable over `GET` and performs the state change | Medium | **MVC 3 only** | §5.4 | 05 |
| F-09-21 | No cross-site request forgery control at all: zero tokens emitted, zero validated, eight unprotected POSTs including sign-in and change-password | High | **MVC 3 only** | §5.6 | 05 |
| F-09-22 | `[Bind(Exclude = "OrderId")]` permits mass assignment of `Total`, and the corrective write targets a different `DbContext` from the one that commits | High | **MVC 3 only** | §5.7 | 05, 12 |
| F-09-23 | The newer editions lost the output encoding MVC 3 applies to the echoed album title; MVC 3's own encoding is for the wrong context | Low | all three | §5.9 | 05 |
| F-09-24 | `AddToCart` mutates the database over `GET`, so no anti-forgery policy can cover it and non-attack fetches trigger it | High | all three | §6.1 | 05 |
| F-09-25 | The cart key doubles as the user's login name, so cart rows are keyed by a non-secret | Medium | all three | §6.2 | 05 |
| F-09-26 | Cart removal is scoped, but the album-title read on the same endpoint is not, so the response discloses a row from any cart | Medium | all three | §6.2 | 05 |
| F-09-27 | Album create and edit bind the entity directly with no input model, and edit performs a full-entity overwrite | Medium | all three | §6.3 | 05 |
| F-09-28 | A single class-level attribute on the persistence entity is the whole over-posting control at checkout, and the promo code bypasses the model entirely | Medium | MVC 5 and MVC 4 | §6.4 | 05, 12 |
| F-09-29 | Nothing requires, redirects to or asserts HTTPS, and no edition emits a single security response header | High | all three | §6.5 | 06, 11 |
| F-09-30 | No `<machineKey>` or key material anywhere, so the keys protecting authentication tickets and anti-forgery tokens belong to the host and the pool identity rather than to the deployment — unpinnable, unrotatable, and not valid on a second host | High | all three | §6.6 | 06, 11 |
| F-09-31 | Every edition registers an initializer that drops and recreates the database holding orders and PII whenever the model changes | **Critical** | all three | §6.7 | 05, 06, 08 |
| F-09-32 | No logging construct of any kind exists, so no authentication, authorization, administrative or order event is recorded. **The catalog that closes it is §6.8.1** — sixteen event classes with identifiers, actors, outcomes, severities and closed field lists | High | all three | §6.8, **§6.8.1** | 03 (emission per producer — thirteen classes in W7, the migration's `AUTHZ-3001` in W8, the three command-produced classes in W12; collection of the thirteen at the W10 hosting gate), 06 (§9.2 sink and privacy policy, §9.5 retention, §8.4 delivery of the `SubjectPseudonym` key and the gate that proves it), 07 (effort and R16) |
| F-09-33 | Nine personal-data fields are stored in the clear with no retention policy, no deletion path and no access log. **The control set that closes it is §6.8.2** — six controls with named approvers, scoped to **three** personal-data stores and not two: the catalog and order store, the Identity and membership store, and the security-event store §6.8.1 introduces, whose `ClientIpAddress` field is personal data the source application never held. A gate satisfied for two of the three does not satisfy this row | High | all three | §6.8, **§6.8.2** | 03 (the approval-gated governance workstream and its two gates), 06 (encryption at rest, backup retention, audit sink), 07 (effort and R15) |
| F-09-34 | All three editions' credential material is tracked in git with the password material intact — two of the three files are the application's own store, and MVC 3's is an **unreferenced, credential-shaped tutorial artifact** that no tracked configuration names and from which **no accounts, hashes or provider selection are inferred** (§5.5, §6.9); the finding is stated on the files' contents, not on which application reads them | **Critical** | all three | §6.9 | 08, 05 |
| F-09-35 | `customErrors` never appears as a live element, so error-detail behaviour is a framework default nobody stated, while every edition ships `debug="true"` | Medium | all three | §6.10 | 05, 06 |
| F-09-36 | **Eleven** external-authentication packages are deployed across two editions — seven in MVC 4, four in MVC 5 — to serve zero enabled providers, with the application code path live. Four of the eleven implement a named provider, which is the narrower unit §6.11 also states | Medium | MVC 5 and MVC 4 | §6.11 | 05, 02 |
| F-09-37 | `DeleteConfirmed` is the one administration action that uses its `Find` result without a null check, producing an unhandled exception with no log | Low | all three | §6.3 | 05 |
| F-09-38 | Sign-in is a username-enumeration oracle by response time: Identity 1.0 returns without any password verification when the named account does not exist, so the identical failure message is a content control only | High | MVC 5 only | §3.4 | 05 |

### 8.1 Distribution

**By severity — a count over the register's Severity column, all 38 rows, no exclusions.** The four
buckets are §1.6's four severity values and there is no fifth; every identifier below appears in
exactly one row of this table, and the four counts sum to the register's row count.
**Both tables below are derived from the register above by the command in this sub-section rather than
counted by hand, and that command reads this file, so editing a register row re-derives them.** An
earlier form of this sub-section published a total of **37** against a 38-row register: F-09-38 was
absent from both tables, and F-09-08 was filed under *MVC 5 only* although its own **Editions** cell
names both newer editions. Two independent errors of one kind — a distribution counted once and then
left alone while the register it summarizes grew — which is why neither table is annotated here and
both are regenerated. Mechanical derivation needs one property of the register to work, and the
register now has it: **the Editions cell of every row is one of exactly seven literals** — `all three`,
`MVC 5 only`, `MVC 5 and MVC 4`, `MVC 4 only`, `MVC 4 and MVC 3`, `MVC 3 only` and the one comparative
value `MVC 5 vs MVC 4` — so the second table is a grouping rather than an interpretation.

```bash
# Every figure in this sub-section, derived from the register table above. Run from the
# repository root. The gsub calls strip the markdown emphasis the cells carry.
DOC=docs/modernization/09-security-assessment.md

# 1. Row count, and that the identifiers run F-09-01 to F-09-38 with no gap and no duplicate:
#    38 distinct values whose minimum is F-09-01 and maximum F-09-38 can only be that range.
awk -F'|' '/^\| F-09-[0-9][0-9] \|/ {print $2}' "$DOC" | tr -d ' ' | sort -u | wc -l
awk -F'|' '/^\| F-09-[0-9][0-9] \|/ {print $2}' "$DOC" | tr -d ' ' | sort -u | sed -n '1p;$p'

# 2. Severity distribution, from the register's third column.
awk -F'|' '/^\| F-09-[0-9][0-9] \|/ {s=$4; gsub(/[* ]/,"",s); print s}' "$DOC" | sort | uniq -c

# 3. Edition distribution, from the register's fourth column.
awk -F'|' '/^\| F-09-[0-9][0-9] \|/ {e=$5; gsub(/\*/,"",e); gsub(/^ +| +$/,"",e); print e}' \
  "$DOC" | sort | uniq -c
```

Output, verbatim, against this file as delivered:

```text
38
F-09-01
F-09-38
      3 Critical
     14 High
      5 Low
     16 Medium
      7 MVC 3 only
      1 MVC 4 and MVC 3
      3 MVC 4 only
      6 MVC 5 and MVC 4
      6 MVC 5 only
      1 MVC 5 vs MVC 4
     14 all three
```

| Severity | Count | Findings |
| --- | --- | --- |
| **Critical** | 3 | F-09-05, F-09-31, F-09-34 |
| **High** | 14 | F-09-03, F-09-08, F-09-13, F-09-16, F-09-17, F-09-18, F-09-21, F-09-22, F-09-24, F-09-29, F-09-30, F-09-32, F-09-33, F-09-38 |
| **Medium** | 16 | F-09-02, F-09-04, F-09-06, F-09-07, F-09-09, F-09-10, F-09-14, F-09-15, F-09-19, F-09-20, F-09-25, F-09-26, F-09-27, F-09-28, F-09-35, F-09-36 |
| **Low** | 5 | F-09-01, F-09-11, F-09-12, F-09-12, F-09-23, F-09-37 |
| **Total** | **38** | — |

3 + 14 + 16 + 5 = 38, which is the register's row count. **The four levels above are the whole scale**
— they are the four §1.6 defines, and every register row carries one of them. No fifth level exists in
this deliverable, so a finding whose interest is comparative rather than exploitable is graded **Low**
against §1.6's own wording ("hardening gap or defence-in-depth loss with no direct exploit path as
shipped") rather than moved off the scale. — is the row that makes the point: it is Low in the
register and Low here.

3 + 14 + 16 + 5 = 38, which is the register's row count.

**By edition — a count over the register's Editions column, all 38 rows, no exclusions.** Seven
buckets are needed rather than four: five editions-sets actually occur, plus one pair that excludes the
migration source and one comparative row, and a four-way split erases the last two rather than counting
them. **No row is excluded from this rollup, including the comparative one** — F-09-12 gets its own
bucket precisely so that it is counted rather than dropped, which is what a hand-kept "all
three / one edition" split did to it.

| Editions the finding holds in | Editions cell | Count | Findings |
| --- | --- | --- | --- |
| All three | `all three` | 14 | F-09-10, F-09-23, F-09-24, F-09-25, F-09-26, F-09-27, F-09-29, F-09-30, F-09-31, F-09-32, F-09-33, F-09-34, F-09-35, F-09-37 |
| MVC 3 only | `MVC 3 only` | 7 | F-09-16, F-09-17, F-09-18, F-09-19, F-09-20, F-09-21, F-09-22 |
| MVC 5 only | `MVC 5 only` | 6 | F-09-03, F-09-04, F-09-06, F-09-07, F-09-09, F-09-38 |
| MVC 5 and MVC 4 | `MVC 5, MVC 4` | 6 | F-09-01, F-09-02, F-09-05, F-09-08, F-09-28, F-09-36 |
| MVC 4 only | `MVC 4 only` | 3 | F-09-13, F-09-14, F-09-15 |
| MVC 4 and MVC 3 — the one pair that excludes the migration source | `MVC 4, MVC 3` | 1 | F-09-11 |
| Comparative — holds in neither edition alone, MVC 5 read against MVC 4 | `MVC 5 vs MVC 4` | 1 | F-09-12 |
| **Total** | — | **38** | — |

14 + 7 + 6 + 6 + 3 + 1 + 1 = 38, which is the register's row count.

**Two rows moved when this rollup was regenerated, and both moves are corrections rather than
reassessments.** F-09-08 is stated in the register for **MVC 5, MVC 4** — five unvalidated
state-changing POSTs at the same line numbers in each of the two newer editions (§3.7, §4.7) — so it
belongs in the pair bucket and not in *MVC 5 only*, where a hand-kept table had placed it while also
omitting it from the pair. F-09-38 was absent from both rollups; it is **MVC 5 only** and **High**, and
it is counted in each. Neither row's severity, editions or evidence changed. Both tables above are generated
from the register's own rows rather than maintained beside them, which is the only way a distribution
stays true as rows are added: a summary edited by hand is a second source of truth, and the row is
what the reader can check.

That MVC 3 carries the most edition-specific findings (**7**) and the migration source the second-most
(**6**) is the concrete form of §2.3's point — the newest edition is not uniformly the safest, and the
migration source is the one whose specific findings the port must carry forward a fix for. The two
exceptional rows are worth keeping visible rather than folded into a remainder: F-09-11 is the only
finding shared by the two Forms-based editions and absent from MVC 5, and F-09-12 is not an
edition-scoped finding at all — it is the observation that comparing the two editions' token
*emission* counts inverts the conclusion, which is why its severity is **Low** and its bucket is
comparative rather than either edition's.

### 8.2 The findings that change what other deliverables can assume

Called out separately because each one invalidates an assumption a downstream deliverable would
otherwise make:

1. **F-09-16 and F-09-17 (MVC 3).** No deliverable may state MVC 3's password or lockout policy, and
   no effort estimate may treat MVC 3's identity configuration as known. The host verification
   deliverable 10 owns is a precondition, not a formality — F-09-17 is High precisely because it
   depends on a host setting nobody has read.
2. **F-09-34 with F-09-05.** The committed credential material is simultaneously a Critical security
   finding and — for **MVC 5's own Identity store**, which is the migration source — the **only**
   authoritative schema evidence the Identity migration deliverable 05 depends on (§6.9). MVC 3's file
   is not that evidence for anything: no tracked configuration names it, and §5.5 infers no account
   material from it, so MVC 3's own identity configuration stays a host-verification item (F-09-16).
   Remediating one blocks the other unless they are sequenced, which is
   deliverable 03's problem and needs to be on its critical path rather than discovered late.
   **The coverage this requirement turns on belongs with F-09-05 and is named here rather than assumed.**
   §3.5's criterion set turns on a single property no store read can establish — that a sign-in attempt
   carrying the published value issues **no authentication cookie** — and
   [05](05-aspnet-core-migration-approach.md)'s §12.4 coverage matrix, which runs to **105** rows, now
   carries **row 105** for it, in the **disposable-store** form §3.5 confines it to: the attempt made
   against a throwaway store, asserting that the response carries no authentication cookie and that the
   counter movement is the probed account's single expected increment and nothing else, after which the
   store is destroyed. A row that ran the same attempt against a live store would not be that assertion
   and is rejected by §3.5's first requirement.
3. **F-09-24 with F-09-08.** The anti-forgery gap and the mutating `GET` are one workstream, not two,
   and the second is not fixable by policy. Any plan that says "add global anti-forgery validation"
   and stops has not addressed `AddToCart` — deliverable 05 must carry the verb change as a separate,
   approval-owned interface delta.
4. **F-09-38 (MVC 5).** No deliverable may treat the identical sign-in failure *message* as evidence
   that the source resists username enumeration, and no coverage assertion may prove the target's
   resistance by comparing two response bodies. The source leaks by *time*, and equalising the work is
   an approved deliberate change rather than a preserved behaviour — so it belongs with the password,
   lockout and cookie values deliverable 05 states and labels as hardening, and its assertion must be a
   timing assertion over a population of attempts. **That assertion has no coverage row yet, and the gap
   is recorded here rather than assumed discharged.**
   [05 §4.3](05-aspnet-core-migration-approach.md) specifies the acceptance test its own
   uniform-verification-cost design needs — a per-population timing-distribution comparison with a stated
   tolerance across **five** account populations rather than one chosen pair — and its §12.4 coverage
   matrix, which runs to **105** rows, now carries **row 104** for it and cites F-09-38 by name. That row
   asserts the distribution comparison across those five populations; a row asserting that two response
   bodies match is not a substitute for it, and treating it as one is the specific mistake this finding
   exists to prevent.

### 8.3 The consumer index — how each row's Consumers value is discharged, in both directions

**The Consumers column is a claim about another document, and a claim about another document is worth
nothing unless a reader can check it from either end.** This sub-section is the inverse of that column:
the column says *which deliverables a row reaches*, and the table below says *which rows reach each
deliverable, and where in that deliverable the row is discharged*. Neither is a second assignment. **The
Consumers column of §8 is the single source and this index is derived from it**, by the script in §9.4 —
so an index that disagrees with the column is the index being wrong, and regenerating it is the repair.

**"Checkable from either end" is a claim about the corpus, so it is *executed* rather than asserted, and
the execution compares pairs rather than counts.** §9.5 builds the expected set of
`(row, consumer)` pairs from the column below and the actual set from the identifiers each consuming
deliverable prints **in the closure section the third column below names for it**, and passes only on
exact set equality. The reason it is a set comparison is that the two failures a hand-kept index
actually produces are both invisible to a count: **a missing pair
compensated by a spurious one**, and **a pair discharged in the wrong deliverable**, each of which leaves
every total unchanged. §9.4 regenerates what this column *says*; §9.5 is the half that reads the ten
consuming deliverables — and it reports a mismatch by **exiting non-zero**, so the claim is enforced
rather than narrated.

**Two traversals, both of which must terminate. This is the property being asserted, and it is what the
index exists to make checkable:**

1. **Register row → discharging item.** Read the row's Consumers cell, look the deliverable up in the
   table below, and go to the closure item named in the third column. That item must reference the row's
   `F-09-nn` identifier, so the landing is unambiguous rather than a topic match.
2. **Consumer's closure item → register row.** Read the `F-09-nn` identifier the consumer's item cites,
   find that row in §8, and confirm the row's Consumers cell names the deliverable you came from. If it
   does not, one of the two is wrong and **§8's cell governs** — a consumer may not acquire a row by
   citing it.

§9.5 traverses both mechanically over the whole corpus: a first-traversal failure appears as a
`MISSING` pair, and a second-traversal failure — a deliverable printing an identifier §8 does not assign
to it, which is the acquisition rule 2 forbids — appears as an `EXTRA` pair.

**The `F-09-nn` identifier is what makes the second traversal terminate, and printing it is the
consumer's obligation rather than this document's.** This index names, per deliverable, the rows it must
discharge and the item where the discharge belongs; what it cannot do is put an identifier into another
deliverable's prose. So where a consumer resolves a row **by topic** — its section says the right thing
about the right control but never writes the row's identifier — the row is discharged in substance and
the *link* is not yet checkable from that end, and adding the identifier is that deliverable's edit. The
third column below is therefore written as the item the identifier belongs on, so a consumer has an
unambiguous target rather than an instruction to find one. **No row is in that state as this document
stands** — §9.5's run finds all 52 pairs printed — and the rule is stated because it is what a `MISSING`
pair from a later run will mean.

**Why the closure lives in the consumer rather than here.** A row's consequence is a decision about the
target, and the target decisions are owned by the deliverables that make them (AAP §0.11.4). If this
document also recorded *how* each row is discharged, there would be two statements of one decision — the
failure this document is strict about elsewhere. So the Consumers column names the destination, the
consumer states the resolution, and this index is only the routing between them.

| Consumer | Rows that reach it | Where the closure is recorded in that deliverable |
| --- | --- | --- |
| **[01](01-architecture-overview.md)** architecture | 1 — F-09-18 | Its own consumer closure: 01 §9.3's capability-gap evidence for MVC 3's administration surface, with the identity stack at 01 §8.3, and the routing row in 01 §11 that names this register row by identifier |
| **[02](02-dependency-inventory.md)** dependencies | 1 — F-09-36 | Its external-authentication package rows — the eleven deployed packages and the four that implement a named provider, which is the unit §6.11 states |
| **[03](03-modernization-roadmap.md)** roadmap | 2 — F-09-32, F-09-33 | Its workstream gates: the emission and collection gates for the event catalog, and the approval-gated personal-data governance workstream whose scope is §6.8.2's **three** stores |
| **[05](05-aspnet-core-migration-approach.md)** ASP.NET Core approach | **28** — F-09-01, F-09-02, F-09-03, F-09-04, F-09-05, F-09-06, F-09-07, F-09-08, F-09-10, F-09-11, F-09-14, F-09-15, F-09-17, F-09-20, F-09-21, F-09-22, F-09-23, F-09-24, F-09-25, F-09-26, F-09-27, F-09-28, F-09-31, F-09-34, F-09-35, F-09-36, F-09-37, F-09-38 | Its target-state policy sections — authentication, cookie, password and lockout values; anti-forgery policy and the `AddToCart` verb change; binding and authorization; the Identity and domain data migrations; the enumeration-timing acceptance test of 05 §4.3. It is the largest consumer because most of this register is a current-state finding whose remedy is a target-state application decision |
| **[06](06-azure-hosting-recommendations.md)** hosting | 8 — F-09-09, F-09-13, F-09-29, F-09-30, F-09-31, F-09-32, F-09-33, F-09-35 | Its transport, header, key-management, observability and audit sections — including the store of record and retention for the security-event store §6.8.2 makes store 3, and the platform side of the pseudonym key |
| **[07](07-effort-risks-sequencing.md)** effort and risk | 4 — F-09-18, F-09-19, F-09-32, F-09-33 | Its effort model and risk register, at the entries the rows' own cells name |
| **[08](08-technical-debt-register.md)** debt | 2 — F-09-31, F-09-34 | Its data-debt entries for the destructive initializer and the committed credential-shaped databases |
| **[10](10-build-and-deployment-requirements.md)** build and deployment | 2 — F-09-16, F-09-17 | Its MVC 3 host-verification requirement, which is the precondition §8.2 item 1 states rather than a formality |
| **[11](11-cloud-readiness-assessment.md)** cloud readiness | 2 — F-09-29, F-09-30 | Its transport and statefulness assessment |
| **[12](12-migration-blockers.md)** migration blockers | 2 — F-09-22, F-09-28 | Its blocker entries for the constructs those rows name |

**52 distinct row-to-consumer references over 37 of the 38 rows.** Both numbers are counts over the
Consumers column and are regenerated by §9.4 rather than maintained here — and the **52 pairs
themselves**, not their count, are what §9.5 compares against the consuming deliverables.

**F-09-12 has no consumer, and that is a stated result rather than a blank.** Its Consumers cell is `—`,
and it is the only row in the register whose cell is. The reason is what the row *is*: F-09-12 records
that comparing MVC 5's and MVC 4's anti-forgery **token-emission** counts inverts the conclusion a
coverage audit would reach, so it is a **methodological warning to a reader of this document** — do not
read emission as coverage — and not a requirement on any target artifact. The requirement it protects is
F-09-08's and F-09-24's, and those two rows carry the consumers. Giving F-09-12 a consumer would create
an obligation no deliverable can discharge, and leaving the cell blank without saying so would read as an
omission; so the cell is an em dash, this paragraph says why, and §9.4's script prints the row by name so
that a *second* consumerless row could never appear silently.

**Two parsing rules, because neither end of a pair reads literally.** Both are live in this corpus, and
either one read wrongly produces a confident report about a corpus that is correct.

1. **A Consumers cell contains identifiers that are not deliverable numbers.** F-09-32's cell names
   roadmap workstreams — `W7`, `W8`, `W10`, `W12` — and F-09-32 and F-09-33 name risk register entries
   `R16` and `R15`. **A `W`-prefixed identifier is a workstream inside deliverable 03 and is never a
   deliverable number**, so `W10` does not make deliverable 10 a consumer of F-09-32, and `W12` does not
   make deliverable 12 one; deliverable 03 is the consumer, and the workstreams are where *within* 03 the
   row lands. §9.4's extraction excludes both prefixed forms and §9.5's excludes them and deletes any
   `§`-reference in the cell as well, for exactly this reason — reading `W10` as `10` is the one
   mis-derivation this column invites. No cell today carries a `§`-reference whose major number is two
   digits, which is why both extractions agree; §9.5 strips them so that adding one later cannot make a
   section reference read as a consumer.
2. **A cited identifier in a consuming deliverable is not automatically a closure.**
   [12](12-migration-blockers.md) §9.1 prints **four** `F-09` identifiers it labels explicitly as
   attributions to the owning deliverable rather than as closures, and **none of those four rows'
   Consumers cells names 12** — which is the same acquisition rule as traversal 2 above, applied by the
   citing document to itself. A parser that read every citation as a closure would report four extra
   pairs against a correct corpus. §9.5 therefore counts **designated closures only**, in two senses:
   it reads each consumer's identifiers out of the closure section the third column above names rather
   than from anywhere in that file, so a passing mention in unrelated prose cannot become a pair; and
   it reads 12's four attributions out of 12's own labelled paragraph rather than from a list kept
   here, validating each exclusion against this column, so excluding a pair the column *does* assign
   fails the run instead of hiding it.

---

## 9. Reproducibility appendix

Every count and every absence claimed above is reproduced here. Commands are run from the repository
root. Those prefixed `git` were run in Git Bash on the Windows host; the PowerShell block below covers
the three measurements git alone cannot make — the binary count and byte total, the byte-wise name
probe, and the version of the MVC assembly under test. §9.3 then records the one compiled harness this
assessment produced — its pinned assembly, its repository impact and the procedure that re-runs it — on
the retained-evidence terms §1.4's fifth convention sets, which is why it carries no harness source, no
build tooling, no literal command lines and no raw console output.

```bash
# --- Authorization enforcement points, per edition (§3.2's rule, §§3.2/4.2/5.2's counts) ----
# Part 1: authorization ATTRIBUTES. [AllowAnonymous] is an opt-out and is excluded by the -w
# 'Authorize' match, which does not match 'AllowAnonymous'.
git grep -now 'Authorize' -- 'src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/*.cs' | wc -l   # 4
git grep -now 'Authorize' -- 'src/MVC4/MvcMusicStore/Controllers/*.cs'                         | wc -l   # 3
git grep -now 'Authorize' -- 'src/MVC5/MvcMusicStore/Controllers/*.cs'                         | wc -l   # 3
# MVC 3's four are two ACTION-level attributes on the ChangePassword pair plus two class-level
# ones; MVC 4's and MVC 5's three are all class-level. §5.2 tabulates the difference, which is
# why the attribute count alone must not be read as coverage.
#
# No edition registers an authorization filter globally, and none appears in a view, so the
# attribute count above is complete:
git grep -n 'Authorize' -- 'src/MVC5/MvcMusicStore/App_Start' 'src/MVC4/MvcMusicStore/App_Start' \
    'src/MVC5/MvcMusicStore/Global.asax.cs' 'src/MVC4/MvcMusicStore/Global.asax.cs' \
    'src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Global.asax.cs' \
    'src/MVC4/MvcMusicStore/Filters' | wc -l                                                             # 0
git grep -n 'Authorize' -- 'src/MVC5/MvcMusicStore/Views' 'src/MVC4/MvcMusicStore/Views' \
    'src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Views' | wc -l                                       # 0
#
# Part 2: IN-ACTION checks -- a client-supplied subject compared against the principal. One in
# MVC 3 and MVC 5 (the order-ownership check), TWO in MVC 4 (that check plus Disassociate's).
git grep -n 'User.Identity.Name'  -- 'src/MVC4/MvcMusicStore/Controllers/AccountController.cs'
#   :126 if (ownerAccount == User.Identity.Name)   <- the comparison; MVC 4's SIXTH point
#   :131 :132 :154 :166 :177 :208 :253 :332 :346   <- the principal used AS THE SUBJECT of a call,
#                                                     not compared against a submitted value
git grep -n 'GetUserId()' -- 'src/MVC5/MvcMusicStore/Controllers/AccountController.cs' | head -1
#   :117 UserManager.RemoveLoginAsync(User.Identity.GetUserId(), new UserLoginInfo(...))
#        MVC 5's equivalent action: the caller's id IS the subject, so there is nothing to compare
#        and nothing to count -- the difference of one between MVC 4 and MVC 5 is real (§4.2).
#
# Part 3: the DATA-SCOPE predicate, counted once per control rather than once per query site.
git grep -c 'CartId == ShoppingCartId' -- 'src/MVC5/MvcMusicStore/Models/ShoppingCart.cs'   # ...:7 sites
# Seven sites, ONE control, evidenced at the removal site [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:64-66] because that is the one
# constraining a mutation against a client-supplied row id. The read on the same endpoint that
# is NOT scoped is F-09-26, in §6.2.
#
# Totals: MVC 5 = 3 + 1 + 1 = 5;  MVC 4 = 3 + 2 + 1 = 6;  MVC 3 = 4 + 1 + 1 = 6.
# Technical Specification §6.4's single figure of five is MVC 5's; §9.2 records the correction.

# --- Anti-forgery: emission vs validation, per edition -----------------------
for e in MVC3 MVC4 MVC5; do printf '%s emission=' $e; git grep -o 'Html.AntiForgeryToken' -- "src/$e/*.cshtml" | wc -l; done
#   MVC3 emission=0    MVC4 emission=8    MVC5 emission=10
git grep -n 'ValidateAntiForgeryToken' -- 'src/MVC3/'      | wc -l   # 0
git grep -n 'ValidateAntiForgeryToken' -- 'src/MVC4/*.cs'  | wc -l   # 7
git grep -n 'ValidateAntiForgeryToken' -- 'src/MVC5/*.cs'  | wc -l   # 8

# --- POST census. Two traps, both live in this repository -------------------
# TRAP 1: the bracketed literal misses `[HttpPost, ActionName("Delete")]`, i.e. the album-delete
#         action. `git grep -c` prints `path:count`, so read the number after the colon.
git grep -c '\[HttpPost\]' -- 'src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs'  # ...:2  <- WRONG
git grep -c 'HttpPost'     -- 'src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs'  # ...:3  <- correct
# TRAP 2: a pathspec of 'src/MVC3/*Controller.cs' also matches the tutorial-asset copy at
#         src/MVC3/MvcMusicStore-Assets/Code/Controllers/AccountController.cs, which holds 3 further
#         POSTs and is not part of any application. Scope each edition to its own project:
git grep -o 'HttpPost' -- 'src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/*.cs' | wc -l   # 8
git grep -o 'HttpPost' -- 'src/MVC4/MvcMusicStore/Controllers/*.cs'                         | wc -l   # 12
git grep -o 'HttpPost' -- 'src/MVC5/MvcMusicStore/Controllers/*.cs'                         | wc -l   # 13
# The unscoped form returns 11 for MVC 3 rather than 8, which is the overcount to avoid.

# --- Transport and response headers: absent everywhere ----------------------
git grep -inE 'RequireHttps|requireSSL' -- 'src/**/*.cs' 'src/**/*.cshtml' 'src/**/*.config' | wc -l   # 0
git grep -inE 'Strict-Transport|customHeaders|X-Frame-Options|Content-Security-Policy|X-Content-Type' \
    -- 'src/**/*.config' 'src/**/*.cs' | wc -l                                                        # 0

# --- Key material and session: absent everywhere ---------------------------
# F-09-30's evidence. Both forms were run; both return 0, and the pathspec is not
# vacuous -- `git grep -in 'debug' -- 'src/**/*.config'` returns 9 over the same set.
git grep -n  'machineKey'   -- '*.config' | grep -v '/packages/'   # no output
git grep -in 'machinekey'   -- '*.config' | grep -v '/packages/' | wc -l    # 0
git grep -in 'sessionState' -- '*.config' | grep -v '/packages/' | wc -l    # 0
git ls-files -- '*.config' '*.Config' | grep -v '/packages/' | grep -v '.nuget/' | wc -l   # 15

# --- Observability: absent everywhere -------------------------------------
git grep -inE 'ILogger|log4net|NLog|Serilog|TraceSource|System\.Diagnostics\.Trace|healthMonitoring|EventLog' \
    -- 'src/**/*.cs' 'src/**/*.cshtml' 'src/**/*.config' | grep -v '/packages/' | wc -l                # 0

# --- customErrors: units matter. 24 word matches, 12 '<customErrors' literals,
#     6 commented example elements, 0 live. See the table in 6.10.
git ls-files -- '*Web.Debug.config' '*Web.Release.config' | grep -v '/packages/' \
  | xargs grep -o 'customErrors'  | wc -l                  # 24  (the WORD, four per file)
git ls-files -- '*Web.Debug.config' '*Web.Release.config' | grep -v '/packages/' \
  | xargs grep -o '<customErrors' | wc -l                  # 12  (two per file: one prose
                                                           #      mention, one element open)
git grep -n  'customErrors' -- 'src/MVC5/MvcMusicStore/Web.config' \
                               'src/MVC4/MvcMusicStore/Web.config' \
                               'src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config' | wc -l      # 0

# --- Exception handling census --------------------------------------------
git grep -n 'AddModelError' -- 'src/**/*.cs' | grep -v '/packages/' | wc -l   # 13 sites
# Exactly one passes an Exception rather than a string:
git grep -n 'AddModelError("", e)' -- 'src/**/*.cs'
#   src/MVC4/MvcMusicStore/Controllers/AccountController.cs:213
git grep -nE '^\s*catch' -- 'src/**/*.cs' | grep -v '/packages/' | wc -l      # 9 catch blocks

# --- Injection and encoding surface: the positive controls ----------------
git grep -nE 'ExecuteSqlCommand|SqlQuery|SqlCommand|CommandText' -- 'src/**/*.cs' | wc -l          # 0
git grep -nE 'Html\.Raw|MvcHtmlString\.Create' -- 'src/**/*.cshtml' 'src/**/*.cs' | wc -l          # 0
git grep -n 'HttpContext.Current' -- 'src/**/*.cs' 'src/**/*.cshtml' | wc -l                       # 0
git grep -n 'HtmlEncode' -- 'src/**/*.cs' 'src/**/*.cshtml'
#   exactly 1: src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/ShoppingCartController.cs:68

# --- MVC 3 has no App_Start folder, hence no provisioning -----------------
test -d src/MVC3/MvcMusicStore-Completed/MvcMusicStore/App_Start && echo present || echo absent   # absent
# -I skips binary files, and it is required here: without it this search reports one match, in
# src/MVC3/MvcMusicStore-Assets/Data/ASPNETDB.MDF, because the classic membership schema ships a
# stored procedure named aspnet_Roles_CreateRole. That is not source code, and it is not MVC 3
# provisioning anything -- it is one more corroboration of F-09-34, visible only because that
# tutorial membership database is committed (§5.5 states what the file is and is not).
git grep -I -inE 'CreateAdminUser|CreateRole|DefaultAdminPassword' -- 'src/MVC3/' | wc -l         # 0

# --- MVC 3's committed membership database is referenced by no configuration ------
# F-09-34's per-edition distinction and §5.5's classification. Nothing in the tree reads it.
git grep -il 'ASPNETDB' -- 'src/' | wc -l                                                        # 0
git ls-files 'src/MVC3/MvcMusicStore-Assets/Data/'      # ASPNETDB.MDF, aspnetdb_log.ldf, the
                                                        # tutorial catalogue pair and the one
                                                        # runnable schema script -- no App_Data
                                                        # folder exists in MVC 3 at all

# --- Dependency-scanning configuration: none ------------------------------
git ls-files | grep -E '^\.github/|dependabot|renovate' | wc -l    # 0
```

```powershell
# --- The 14 committed database binaries and their exact total -------------
$f = git ls-files | Where-Object { $_ -match '\.(mdf|ldf)$' -or $_ -match '\.(MDF|LDF)$' }
$f.Count                                                                       # 14
($f | ForEach-Object { (Get-Item $_).Length } | Measure-Object -Sum).Sum        # 43376640

# --- Committed-database name probe (EVIDENCE, NOT PROOF) -----------------
# A name probe cannot distinguish an absent column from one stored in a form the
# probe does not surface, so every conclusion drawn from it is qualified in the
# text (F-09-03, F-09-34). Two properties of the probe matter and are built in
# rather than assumed:

#   1. Names are searched as UTF-16LE, because SQL Server stores object names as
#      nvarchar; an ASCII-only search finds nothing at all.
#   2. The search is BYTE-WISE, not one alignment. A UTF-16LE byte pattern can
#      begin at an even or an odd byte offset, and decoding once from offset 0
#      surfaces only the even-aligned occurrences -- which for three of the names
#      probed below is NOT the first occurrence in the file. Decoding a second time from
#      offset 1 covers the odd alignment, and every occurrence starts at one or
#      the other, so the lower of the two offsets is the true first occurrence.
#      Every offset published in this document is that value.
function Probe($path, $patterns) {
  $bytes = [System.IO.File]::ReadAllBytes($path)
  $even = [System.Text.Encoding]::Unicode.GetString($bytes, 0, $bytes.Length - ($bytes.Length % 2))
  $odd  = [System.Text.Encoding]::Unicode.GetString($bytes, 1, $bytes.Length - 1 - (($bytes.Length - 1) % 2))
  foreach ($p in $patterns) {
    $ie = $even.IndexOf($p); $io = $odd.IndexOf($p)
    $oe = if ($ie -ge 0) { 2 * $ie }     else { [int]::MaxValue }   # byte offset of an even-aligned hit
    $oo = if ($io -ge 0) { 1 + 2 * $io } else { [int]::MaxValue }   # byte offset of an odd-aligned hit
    $off = [Math]::Min($oe, $oo)
    $txt = if ($off -eq [int]::MaxValue) { 'not found at any alignment' } else { '0x{0:X}' -f $off }
    "{0,-20} present={1,-6} first-byte-offset={2}" -f $p, ($off -ne [int]::MaxValue), $txt
  }
}
Probe 'src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205.mdf' `
      @('AspNetUsers','AspNetRoles','PasswordHash','SecurityStamp','Administrator',
        'LockoutEnabled','LockoutEndDateUtc','AccessFailedCount','TwoFactorEnabled','EmailConfirmed')
#   AspNetUsers          present=True   first-byte-offset=0xECBF2
#   AspNetRoles          present=True   first-byte-offset=0xECB9C
#   PasswordHash         present=True   first-byte-offset=0x732E9   <- odd-aligned; the
#                                       even-only decode would report 0x138FFE instead
#   SecurityStamp        present=True   first-byte-offset=0x73336
#   Administrator        present=True   first-byte-offset=0x3700B   <- odd-aligned; the
#                                       even-only decode would report 0x3704A instead
#   LockoutEnabled       present=False  first-byte-offset=not found at any alignment
#   LockoutEndDateUtc    present=False  first-byte-offset=not found at any alignment
#   AccessFailedCount    present=False  first-byte-offset=not found at any alignment
#   TwoFactorEnabled     present=False  first-byte-offset=not found at any alignment
#   EmailConfirmed       present=False  first-byte-offset=not found at any alignment
#   The five absent names are absent at BOTH alignments, which is the whole of what
#   this probe supports. F-09-03 does NOT rest on it: that finding rests on the
#   sign-in code path and the pinned Identity generation, and the physical column
#   status of the shipped store stays gated on an authoritative sys.columns
#   extraction -- because absence from a name probe is evidence and not proof.
Probe 'src/MVC4/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20120831200627.mdf' `
      @('webpages_Membership','webpages_Roles','UserProfile','Password','Administrator')
#   webpages_Membership  present=True   first-byte-offset=0xEDAD0
#   webpages_Roles       present=True   first-byte-offset=0xEDBE2
#   UserProfile          present=True   first-byte-offset=0xED9DA
#   Password             present=True   first-byte-offset=0x73823   <- odd-aligned; the
#                                       even-only decode would report 0x7387E instead
#   Administrator        present=True   first-byte-offset=0x3AAA0
Probe 'src/MVC3/MvcMusicStore-Assets/Data/ASPNETDB.MDF' `
      @('aspnet_Membership','aspnet_Users','aspnet_Roles','Password','PasswordSalt')
#   aspnet_Membership    present=True   first-byte-offset=0x1C4AE
#   aspnet_Users         present=True   first-byte-offset=0xE8F62
#   aspnet_Roles         present=True   first-byte-offset=0x1C700
#   Password             present=True   first-byte-offset=0x5485E
#   PasswordSalt         present=True   first-byte-offset=0x548F4
#   All five are even-aligned here, so this edition's offsets are unaffected by the
#   alignment question -- but they were measured with the same byte-wise probe.

# --- F-09-14 / F-09-15: what Html.ValidationSummary actually renders -----
# Compiled and run against MVC 4's OWN committed assembly, so the result is this
# edition's behaviour and not a guess about some version of MVC. The pinned assembly,
# the repository impact and the reproduction procedure are in 9.3; the harness source,
# the build tooling, the command lines and the raw console output are NOT retained
# (1.4, fifth convention), and 4.9 carries the transcribed result the run bounded the
# finding with. The one line below is an ordinary file read, not part of the harness.
$mvc = 'src/MVC4/MvcMusicStore/packages/Microsoft.AspNet.Mvc.4.0.20710.0/lib/net40/System.Web.Mvc.dll'
(Get-Item $mvc).VersionInfo.FileVersion                                        # 4.0.20710.0
```

### 9.1 The constraint this work was held to

§1.3 states the constraint and the criterion it is checked against, and the tracked outcome meets it:
thirteen additions under `docs/modernization/`, and no existing file modified, deleted or renamed.
Every command published in this document reads the repository and writes nothing into it — including
`git clean -ndX` below, which is a dry run. The one compiled artefact this assessment produced — the
`ValidationSummary` harness §9.3 records — was built and run in a scratch directory outside the
checkout, reading MVC 4's committed package payload and writing nothing into it, and §9.3 records that
impact and the procedure that re-establishes it for exactly that reason. Per §1.4's fifth convention it
does **not** retain the harness source, the build tooling, the command lines or the raw output, so the
claim a reader checks here is the *repository impact*, which is the only part of a scratch-directory
build this document is in a position to attest.

**An earlier version of this section claimed more than that, and the additional claim was false.** It
said that nothing at all had been written into the checkout. Package restores and build attempts
*were* run against the three editions while the assessment was being made — deliverable 10 owns what
they established about each edition — and they wrote eight gitignored trees into this working tree. A
restore cannot be redirected outside the tree the way a compiler invocation can, because the projects
resolve their references through `..\packages` hint paths
[src/MVC4/MvcMusicStore/MvcMusicStore.csproj:66] and
[src/MVC5/MvcMusicStore/MvcMusicStore.csproj:66], which is why two of the eight landed at
`src/MVC4/packages` and `src/MVC5/packages` rather than beside the project:

| Tree written into the checkout | Files |
| --- | --- |
| `src/MVC3/MvcMusicStore-Completed/MvcMusicStore/bin` | 5 |
| `src/MVC3/MvcMusicStore-Completed/MvcMusicStore/obj` | 7 |
| `src/MVC4/MvcMusicStore/bin` | 51 |
| `src/MVC4/MvcMusicStore/obj` | 7 |
| `src/MVC4/packages` | 231 |
| `src/MVC5/MvcMusicStore/bin` | 52 |
| `src/MVC5/MvcMusicStore/obj` | 7 |
| `src/MVC5/packages` | 167 |
| **Total** | **527 files, 114,310,394 bytes** |

All eight have since been removed and their absence verified, by checks 2 and 3 below. Those counts
are the measurement taken at removal time and are deliberately **not** reproducible from the checkout
now — the state they describe no longer exists, which is the reason for recording it here rather than
letting a passing check imply it never happened.

**Why the evidence this section originally published could not have caught it.** `bin/` and `obj/`
are ignored at [.gitignore:1-2]. The two restored `packages` trees are matched by exactly one rule,
and it is not the obvious one: `Packages/` at [.gitignore:33], whose only separator is trailing, so
it matches a directory of that name at **any** depth — and it matches these lowercase directories
only because this host reports `core.ignorecase = true` —
`git check-ignore -v --no-index src/MVC4/packages/x src/MVC5/packages/x` names
`.gitignore:33:Packages/` for both. `packages/*` at [.gitignore:15] never covered them: an interior
separator anchors a pattern to the directory holding the `.gitignore`, which is the repository root,
so it matches `packages/x` there and nothing nested.
[04 — .NET 8 migration strategy](04-dotnet8-migration-strategy.md) §A.6 owns that rule analysis with
its own probe, and it is cross-referenced rather than restated in a third form. On this checkout,
then, all eight trees were ignored — and `git status --porcelain` does not report ignored files, while
a tracked-file diff reads two commits and never inspects the working tree at all. Both published
checks therefore printed precisely what a clean checkout prints while 114 MB of restored package
payload and build output sat in that checkout. The output itself was recoverable and is gone; the
defect was an acceptance check that could not observe the thing it was cited as proving — the same
distinction §3.7 draws when it refuses to read an emitted anti-forgery token as evidence that a token
is validated, applied here to this document's own attestation.

**The case dependency sharpens that defect rather than excusing it**, which is why a security
document states it rather than leaving it in a locator. On a case-sensitive checkout **no rule
matches `src/MVC4/packages` or `src/MVC5/packages` at all** —
`git -c core.ignorecase=false check-ignore -v --no-index --non-matching src/MVC4/packages/x src/MVC5/packages/x`
returns `::` for both paths and exits 1 — so on such a host the very same published check 1 would
have reported those two trees, holding 398 of the 527 files, as untracked rather than printing
nothing, and the omission would have been caught. A control whose ability to observe what it attests
varies with the host filesystem's casing behaviour is weaker than one that does not, and this is a
small, concrete instance of it. Check 2 below is the one that does not vary: it reports such a tree
as ignored where the casing rule reaches it and as untracked where no rule does, which is exactly the
coverage check 1 lacks.

**The attestation, as four checks that hold together or not at all.** The first two ask the same
question of tracked and of ignored files, the third confirms the second from the opposite direction,
and the fourth states *what* was added rather than only that nothing is pending. That is the
criterion AAP §0.11.5 makes final, carrying the ignored-file coverage it needs in order to mean what
it says:

**Check 4 is the only commit range anywhere in this document, and both of its ends are named here so
that neither has to be inferred.** Its left side is the **immutable evidence revision**
`ea2552d6eda7c20e9477a512e5c615665618cf35`, the pre-assessment commit of the repository as this
assessment read it — every `[<path>:<locator>]` citation in this document resolves against `ea2552d`,
and it does not move. Its right side, `HEAD`, is the **delivery commit the reviewer has checked out**:
the commit that carries the thirteen deliverables, which is what makes the range read "what this
assessment added" rather than "what has happened since". The three command lines below spell the left
side in full rather than as `ea2552d` for one reason only — they are meant to be copied and run
verbatim.

```bash
# All four are read together. Any one of them passing in isolation proves nothing, and
# checks 1 and 4 passing in isolation is exactly the gap described above.

# 1. Tracked working tree: nothing added, modified, deleted or staged.
git status --porcelain                       # 0 lines

# 2. The same question with ignored files included -- the only one of the four that can
#    observe restored package payload or build output, because .gitignore hides those
#    from check 1.
git status --porcelain --ignored             # 0 lines

# 3. Confirmation from the opposite direction: what git would delete if it were asked to
#    remove ignored files. -n dry-runs it, -d covers whole directories, -X restricts it
#    to ignored paths; as written it removes nothing. It listed the eight trees tabulated
#    above for as long as they existed, and prints nothing now that they are gone.
git clean -ndX                               # no output

# 4. What this assessment added, measured from the pre-assessment baseline commit.
#    Exactly thirteen additions, all under docs/modernization/, and nothing else.
#    Both ends of this range are named deliberately, and neither is a placeholder.
#    LEFT  ea2552d6eda7c20e9477a512e5c615665618cf35 -- the immutable pre-assessment
#          revision: the last commit before this assessment began, and the revision at
#          which every [<path>:<locator>] citation in this document resolves. It is
#          spelled in full so the line can be copied and run verbatim, and it does not
#          move.
#    RIGHT HEAD -- named as HEAD rather than as a hash because a document cannot cite
#          the commit that creates it, and a literal hash written here would be false at
#          the next commit. The reader resolves HEAD on the delivered checkout, which is
#          the commit carrying the thirteen deliverables; that is what makes the range
#          read "what this assessment added" rather than "what has happened since".
git diff --name-status ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD
#   A  docs/modernization/01-architecture-overview.md
#   A  docs/modernization/02-dependency-inventory.md
#   A  docs/modernization/03-modernization-roadmap.md
#   A  docs/modernization/04-dotnet8-migration-strategy.md
#   A  docs/modernization/05-aspnet-core-migration-approach.md
#   A  docs/modernization/06-azure-hosting-recommendations.md
#   A  docs/modernization/07-effort-risks-sequencing.md
#   A  docs/modernization/08-technical-debt-register.md
#   A  docs/modernization/09-security-assessment.md
#   A  docs/modernization/10-build-and-deployment-requirements.md
#   A  docs/modernization/11-cloud-readiness-assessment.md
#   A  docs/modernization/12-migration-blockers.md
#   A  docs/modernization/README.md
git diff --name-status ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD | grep -c '^A'          # 13
git diff --name-status ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD \
  | grep -v 'docs/modernization/' | wc -l                                                     # 0
# No M row, no D row, no R row: no existing file was modified, deleted or renamed. This
# check compares two commits, so it is silent about anything sitting untracked or ignored
# in the working tree -- that is what checks 2 and 3 are for, and it is why publishing this
# one beside a bare check 1 attested something neither of them had actually looked at.
```

**Why check 4's range has one pinned end and one symbolic one.** The **start** is pinned, in full, to
`ea2552d6eda7c20e9477a512e5c615665618cf35` — the last commit before this assessment began — because a
baseline that moves would let the diff shrink as work landed and report a clean result for the wrong
reason. The **end** is `HEAD` rather than a literal, and that is a property of the artifact rather than a
looseness in the evidence: a commit's hash is computed over its own content, this file included, so **no
document can contain the hash of the commit that adds it**. Any literal written in `HEAD`'s place would
necessarily name some earlier commit, and would therefore assert a tree that is not the one under review.
A reader who wants both ends pinned for their own records resolves the tip once on the checkout in front
of them — `git rev-parse HEAD` — and substitutes the result wherever `HEAD` appears above. That
substitution changes the notation and **none** of check 4's four expected values: thirteen rows, thirteen
of them `A`, zero carrying any other status, and zero outside `docs/modernization/`. Each of those is a
property of the range, not of the tip.

**The standard the restores should have been held to is already in this document.** §9.3's harness
copied MVC 4's `net40` payload into a scratch directory, compiled there and ran there, which is why
its provenance row can record *Repository impact: none* and have that be checkable rather than
asserted. The restores were run in place instead, and nothing this document published was looking for
what they left. Either practice is available to a later phase — build and restore against a throwaway
copy of the tree, or work in the tree and make checks 2 and 3 part of the acceptance set rather than
an afterthought — and it is only the combination of an in-place build with a check blind to its output
that is not.

### 9.2 Secondary cross-reference

Technical Specification §6.4 describes the same three parallel authentication stacks, the same
inheritance of password and session policy from framework defaults, the same per-edition anti-forgery
coverage, and the same verified absence of audit logging, application cryptography and transport
protection. It corroborates this assessment and is cited nowhere in place of the repository evidence
that establishes each finding.

**One point of disagreement, recorded rather than smoothed over, because AAP §0.1.3 makes the
repository govern.** Specification §6.4 states **five** authorization enforcement points as a property
of the application. That figure is right for **MVC 5** and wrong for the other two editions: MVC 4 has
**six** — the specification's five plus `Disassociate`'s owner comparison
[src/MVC4/MvcMusicStore/Controllers/AccountController.cs:126] — and MVC 3 has **six** differently
composed (§5.2). An earlier form of this document inherited the specification's single figure and
published it as the application's, which is the failure mode the citation policy exists to prevent: a
secondary source read as if it were the primary one. §3.2 now states the counting rule and the three
per-edition figures from the repository, §4.2 and §5.2 enumerate MVC 4's and MVC 3's in full, and this
paragraph is the correction to §6.4 rather than a second reading of it.

Where this document and any other input disagree, the repository governs — which is why §4.9 states
the disclosure finding the repository establishes and additionally records what the shipped assembly
renders as a bound on it, rather than letting either half stand in for the other, and why §5.1
declines to state a password policy that the checkout does not contain.

### 9.3 The MVC 4 `ValidationSummary` harness — what it measured, and how to re-run it

§4.9 reports what MVC 4's own `Html.ValidationSummary` does with an `Exception` in model state, and
records it as a **transcribed observation of one run** rather than as a durable property of the
framework. This sub-section is the other half of that record: the pinned assembly the run exercised,
the fact that it touched nothing in the checkout, and the procedure by which a reader re-establishes
the same bound. It is the one measurement in this assessment that could not be made by reading a file,
which is why it is the only one with a harness at all.

**The evidence policy is §1.4's fifth convention, and this sub-section applies it rather than stating a
second one.** An earlier form of §9.3 published the harness source in full, the compiler and host
versions, the literal build and run command lines and the verbatim console output — while §1.4 and §4.9
both declared those four things **not retained**. That was one measurement under two policies, and the
published form was the one neither of the other two sections claimed. The retained form is the one
below: **what was measured, the assembly identity it was measured against, the repository impact, and
the reproduction procedure.** The four unretained things are **deleted rather than annotated**. Deleting
them costs nothing a reader can use: §4.9 already states the transcribed result, a re-run re-establishes
it better than a transcript can, and a console listing inside a committed document is one more artifact
no reader is able to verify.

**Provenance.**

| | |
| --- | --- |
| What it measures | `ModelError.ErrorMessage` and the rendered output of `Html.ValidationSummary()` and `Html.ValidationSummary(true)` when model state carries an `Exception` rather than a string |
| Assembly under test | `src/MVC4/MvcMusicStore/packages/Microsoft.AspNet.Mvc.4.0.20710.0/lib/net40/System.Web.Mvc.dll`, assembly version 4.0.0.0, file version 4.0.20710.0 — printed by the harness as its own first output line, so the run identified the assembly it exercised rather than assuming it |
| Date run | **2026-08-28.** The compiler and host versions are deliberately **not** recorded, per §1.4's fifth convention: this document cannot make the run re-executable, so a tool version here would read as provenance while establishing nothing a reader could check |
| Repository impact | **None.** The source, the built executable and the copied package payload all lived outside the checkout, in a scratch directory; `packages/` was read and never written; `git status --porcelain` reported no entry under `src/` before or after the run, so no path the harness touched changed |

**How to re-run it, which is the durable form of this measurement.** §1.4's fifth convention retains
the reproduction procedure rather than the run, so what follows is what a reader executes to
re-establish the bound for themselves. Four steps, and each one is stated because each one can be got
wrong in a way that produces a confident wrong answer:

- **Copy MVC 4's committed `net40` package payload into a scratch directory outside this checkout, and
  build and run there.** The payload is read and never written, and no harness file ever exists inside
  the repository — which is what makes the *Repository impact* row above checkable rather than merely
  asserted, and it is the practice §9.1 contrasts the in-place restores against.
- **Copy the whole `net40` set, not only `System.Web.Mvc.dll`.** `System.Web.Mvc` 4.0 loads
  `System.Web.WebPages` 2.0.0.0 from the `HtmlHelper` constructor, so a partial copy fails with a
  missing-assembly error instead of rendering something misleading. That is the safe failure mode, and
  it is the reason the step is called out rather than left to inference.
- **Compile a console program against `System.Web.dll` and the copied `System.Web.Mvc.dll`** that
  builds an `HtmlHelper` over a `ViewDataDictionary`, adds one model-state error through the
  **`Exception`** overload of `AddModelError` — the overload `AccountController.cs:213` uses — and
  prints four things: `ModelError.ErrorMessage`, whether `ModelError.Exception` survived, and the
  rendered output of both `Html.ValidationSummary()` and `Html.ValidationSummary(true)`. Print the
  loaded assembly's version and file version first, so the run identifies its own subject.
- **Include the control: a second helper whose model-state error is added through the `string`
  overload.** Without it, a run that renders an empty list item is indistinguishable from a stub that
  renders nothing at all, and the empty output would mean nothing. The control's outcome is the last
  row of §4.9's results table for exactly that reason.

A reader who needs the figures as evidence rather than as a bound re-runs that procedure and retains
**their own** run's complete output — source, tool versions, command lines and raw console output — as
a build artifact outside this repository, which is what §1.4's fifth convention requires of any
implementation that depends on the measurement.

**What the harness does and does not establish.** It establishes what this edition's own helper emits
for this model-state shape, which is what F-09-15 rests on, and it establishes by its control line
that the measurement is real rather than a stub returning empty output. It does **not** establish
anything about a custom validation summary, a JSON serialization of model state, or a different
framework version — which is precisely why F-09-14 is recorded as a live hazard rather than closed by
this result. The finding that the exception object is placed on a response-bound channel does not
depend on the harness at all: it rests on
[src/MVC4/MvcMusicStore/Controllers/AccountController.cs:211-213] and on the rendering chain cited in
§4.9.

### 9.4 Regenerating §8.1's rollups and §8.3's index from the register

**Every derived number about the register is a count over the register, and this is the script that
produces them.** It is published for the same reason the other commands in this section are: a rollup
maintained by hand drifts from its source silently, and the only durable defence is that regenerating
it is cheaper than editing it. Run from the repository root; it reads §8's table and nothing else.

**Reading this document and nothing else is also this script's limit, and that limit is why §9.5
exists.** Everything below is derived from §8's rows, so it establishes what the register *assigns* —
including §8.3's index and the consumer counts inside it — and it cannot establish that a consuming
deliverable carries the other end of any assignment, because it never opens one. §9.5 is that half, and
it compares the pairs rather than the counts this script prints.

```powershell
# Parses the 38 register rows out of §8 and re-derives, from the rows alone:
#   1. identifier contiguity F-09-01..F-09-38 (no gap, no reuse)
#   2. §8.1's severity rollup, over §1.6's four values and no fifth
#   3. §8.1's edition rollup, by exact match on the seven canonical Editions strings
#   4. §8.3's consumer inverse index, and the rows that have no consumer
# The Editions and Severity columns are read with emphasis markers stripped; the Consumers
# column is read verbatim, because §8.3's index is an inverse of it and must not normalize it.
$doc  = 'docs/modernization/09-security-assessment.md'
$rows = Get-Content -Encoding UTF8 -LiteralPath $doc |
        Where-Object { $_ -match '^\|\s*F-09-\d\d\s*\|' } |
        ForEach-Object {
          $c = $_ -split '\|'
          [pscustomobject]@{ Id = $c[1].Trim()
                             Severity  = ($c[3].Trim() -replace '\*','')
                             Editions  = ($c[4].Trim() -replace '\*','')
                             Consumers =  $c[6].Trim() } }

'rows = {0}' -f @($rows).Count                                     # expect 38
'contiguous = {0}' -f ((($rows.Id) -join ',') -eq
    (((1..38) | ForEach-Object { 'F-09-{0:d2}' -f $_ }) -join ',')) # expect True

@('Critical','High','Medium','Low') | ForEach-Object {
  $g = @($rows | Where-Object Severity -eq $_)
  '{0,-8} {1,3}  {2}' -f $_, $g.Count, (($g.Id) -join ', ') }      # expect 3 / 14 / 16 / 5

@('all three','MVC 3 only','MVC 4 only','MVC 5 only',
  'MVC 5, MVC 4','MVC 4, MVC 3','MVC 5 vs MVC 4') | ForEach-Object {
  $g = @($rows | Where-Object Editions -eq $_)
  '{0,-14} {1,3}  {2}' -f $_, $g.Count, (($g.Id) -join ', ') }     # expect 14/7/3/6/6/1/1

# Any row whose Editions cell is not one of those seven, or whose Severity is not one of those
# four, prints here. The correct output is nothing at all.
$rows | Where-Object { @('all three','MVC 3 only','MVC 4 only','MVC 5 only',
                         'MVC 5, MVC 4','MVC 4, MVC 3','MVC 5 vs MVC 4') -notcontains $_.Editions -or
                       @('Critical','High','Medium','Low') -notcontains $_.Severity } |
  ForEach-Object { 'NON-CANONICAL: {0} [{1}] [{2}]' -f $_.Id, $_.Severity, $_.Editions }

# §8.3's inverse index. Two-digit deliverable numbers only, so that a workstream identifier such
# as W10 inside a Consumers cell is never mistaken for deliverable 10 -- the exact confusion §8.3
# warns about.
$byConsumer = @{}
foreach($r in $rows){
  if($r.Consumers -eq [char]0x2014){ continue }
  foreach($m in [regex]::Matches($r.Consumers,'(?<![\w-])(0[1-9]|1[0-2])(?![\w-])')){
    if(-not $byConsumer.ContainsKey($m.Value)){ $byConsumer[$m.Value] = @() }
    if($byConsumer[$m.Value] -notcontains $r.Id){ $byConsumer[$m.Value] += $r.Id } } }
$byConsumer.Keys | Sort-Object | ForEach-Object {
  '-> {0}: {1,2}  {2}' -f $_, @($byConsumer[$_]).Count, ($byConsumer[$_] -join ', ') }
$rows | Where-Object { $_.Consumers -eq [char]0x2014 } |
  ForEach-Object { 'NO CONSUMER: {0}' -f $_.Id }                   # expect F-09-12, and only it
```

Verbatim output at the time of writing, which is the state §8.1 and §8.3 publish:

```text
rows = 38
contiguous = True
Critical   3  F-09-05, F-09-31, F-09-34
High      14  F-09-03, F-09-08, F-09-13, F-09-16, F-09-17, F-09-18, F-09-21, F-09-22, F-09-24,
              F-09-29, F-09-30, F-09-32, F-09-33, F-09-38
Medium    16  F-09-02, F-09-04, F-09-06, F-09-07, F-09-09, F-09-10, F-09-14, F-09-15, F-09-19,
              F-09-20, F-09-25, F-09-26, F-09-27, F-09-28, F-09-35, F-09-36
Low        5  F-09-01, F-09-11, F-09-12, F-09-23, F-09-37
all three       14  F-09-10, F-09-23, F-09-24, F-09-25, F-09-26, F-09-27, F-09-29, F-09-30,
                    F-09-31, F-09-32, F-09-33, F-09-34, F-09-35, F-09-37
MVC 3 only       7  F-09-16, F-09-17, F-09-18, F-09-19, F-09-20, F-09-21, F-09-22
MVC 4 only       3  F-09-13, F-09-14, F-09-15
MVC 5 only       6  F-09-03, F-09-04, F-09-06, F-09-07, F-09-09, F-09-38
MVC 5, MVC 4     6  F-09-01, F-09-02, F-09-05, F-09-08, F-09-28, F-09-36
MVC 4, MVC 3     1  F-09-11
MVC 5 vs MVC 4   1  F-09-12
-> 01:  1  F-09-18
-> 02:  1  F-09-36
-> 03:  2  F-09-32, F-09-33
-> 05: 28  F-09-01, F-09-02, F-09-03, F-09-04, F-09-05, F-09-06, F-09-07, F-09-08, F-09-10,
           F-09-11, F-09-14, F-09-15, F-09-17, F-09-20, F-09-21, F-09-22, F-09-23, F-09-24,
           F-09-25, F-09-26, F-09-27, F-09-28, F-09-31, F-09-34, F-09-35, F-09-36, F-09-37,
           F-09-38
-> 06:  8  F-09-09, F-09-13, F-09-29, F-09-30, F-09-31, F-09-32, F-09-33, F-09-35
-> 07:  4  F-09-18, F-09-19, F-09-32, F-09-33
-> 08:  2  F-09-31, F-09-34
-> 10:  2  F-09-16, F-09-17
-> 11:  2  F-09-29, F-09-30
-> 12:  2  F-09-22, F-09-28
NO CONSUMER: F-09-12
```

The identifier lists are wrapped for width in the block above and are single lines in the real
output; nothing else is reformatted. `NON-CANONICAL:` printed no line, which is the check that makes
the two rollups exact-match groupings rather than readings of prose.

### 9.5 The pair-set check — §8.3's two traversals, executed across the corpus

**§8.3 claims its index is checkable from either end; this is the command that checks it, and it
compares sets of pairs rather than counts.** The expected set is every `(row, consumer)` pair in §8's
Consumers column. The actual set is every pair `(identifier, deliverable)` a consuming deliverable
prints **inside the closure section §8.3's third column names for it** — the reciprocal-closure
structures those deliverables carry for this purpose — and never elsewhere in the file, because a
passing mention in unrelated prose is not a discharge and must not be able to manufacture a pair. The
run passes **only on exact set equality**, because the two failures a routing index actually produces
are both invisible to a count that matches: **a missing pair compensated by a spurious one**, and **a
pair discharged in the wrong deliverable**. Both leave every total unchanged, and the negative control
below is one of them.

**The two parsing rules of §8.3 are what make the two sets mean what they say, and getting either wrong
produces a confident report about a corpus that is correct.** A `W`-prefixed token such as `W10` or
`W12` is a **workstream inside deliverable 03** and never deliverable 10 or 12, so the tokenizer keeps
letters and `-` inside a token and deletes `§`-references before matching; and a cited identifier is not
automatically a closure, so the **four** identifiers 12 §9.1 labels as attributions to their owner are
read out of 12's own labelled paragraph and excluded. **Neither exclusion is trusted**: every excluded
pair is checked against §8's column, and one the column does assign is reported as `INVALID EXCLUSION`
*and* as a `MISSING` pair, so an exclusion cannot mask a real gap. The marker sentence in 12 is the
contract — if that paragraph is ever reworded, its four identifiers stop being excluded and the run
reports four extras, which is a loud failure rather than a silent pass.

**The exit status is part of the check and the block is written that way deliberately.** A gate that
prints `FAIL` and exits `0` cannot stop a pipeline, so its output is advisory rather than enforcing —
which is the one thing this block must not be, since §8.3's "checkable from either end" claim rests on
it. So **every** mismatch path — `MISSING`, `EXTRA`, `MISPLACED` and `INVALID EXCLUSION` — exits **1**,
every internal error exits **3**, and the pass path is the only path that exits **0**. The three exits
are stated in the block's own header so that a later reader does not read the `exit` lines as
defensive noise and simplify them away.

**Internal errors fail closed rather than degrading into a weaker check**, and that is the second half
of the same property. The block reads each consumer's closure section through a small map of
`<deliverable> <file>|<heading marker>` rows, one per deliverable §8's column names, and it exits **3**
rather than continuing when a file cannot be read, when §8's register parses to nothing, when a marker
does not match exactly one heading in the file it names, when a designated closure section prints no
identifier at all, or when §8 assigns a pair to a deliverable the map does not carry. Two of those are
worth naming, because both are states a reworded corpus reaches: a consumer that renames its closure
section makes the run stop with a named error instead of falling back to a whole-file scan, and a row
routed to `04` or the README — the two deliverables that carry no closure section, because no row names
them — stops the run rather than being reported as an ordinary `MISSING` pair against a section that
was never read.

```bash
# 09 §8.3 claims its consumer index is checkable from either end. This is that check, and it is a
# comparison of SETS OF PAIRS rather than of counts: equal counts cannot detect a missing pair
# compensated by an extra one, nor a pair that landed in the wrong consumer. Run from the
# repository root. It reads this document and the ten consumers §8's column names, and writes only
# inside a temporary directory.
#
# THE EXIT STATUS IS PART OF THE CHECK AND IS DELIBERATE. Do not "simplify" any path below back
# into a bare message: a gate that prints FAIL and exits 0 cannot fail a pipeline, so its evidence
# is advisory rather than enforcing, which is the one thing this block must not be.
#   0  the two pair sets are identical and every exclusion was validated -- the ONLY exit-0 path
#   1  a mismatch: MISSING, EXTRA, MISPLACED or INVALID EXCLUSION
#   3  an internal error: an unreadable file, a register that does not parse, an empty expected
#      set, a designated closure section that is missing or names no identifier, or a consumer
#      that §8 assigns a pair to and this block holds no closure section for
# `pipefail` is set for the same reason: without it a failing extraction at the head of a pipeline
# is masked by the exit status of the `sort` at its tail.
set -u -o pipefail
die() { printf 'INTERNAL ERROR: %s\n' "$*" >&2; exit 3; }
w=$(mktemp -d) || die 'cannot create a temporary directory'
trap 'rm -rf "$w"' EXIT
cd docs/modernization || die 'run this from the repository root'
[ -r 09-security-assessment.md ] || die '09-security-assessment.md cannot be read'

# --- 1. EXPECTED pairs: (row, consumer) read from §8's sixth column, the single source ---------
awk -F'|' '
  /^\|[[:space:]]*F-09-[0-9][0-9][[:space:]]*\|/ {
    id = $2; gsub(/[[:space:]]/, "", id)
    cell = $7; gsub(/§[0-9]+(\.[0-9]+)*/, " ", cell)
    n = split(cell, tok, /[^A-Za-z0-9-]+/)
    for (i = 1; i <= n; i++) if (tok[i] ~ /^(0[1-9]|1[0-2])$/) print id, tok[i]
  }' 09-security-assessment.md | sort -u > "$w/expected" || die "§8's register did not parse"
[ -s "$w/expected" ] || die "the expected set is empty -- §8's register parsed to nothing"

# --- 2. ACTUAL: each consumer's DESIGNATED closure section, and nothing else ------------------
# An identifier is a closure only where the consumer designates it as one, so the actual set is
# read from the reciprocal-closure section §8.3's third column names in each consumer -- never
# from anywhere else in the file, because a passing mention in unrelated prose is not a discharge
# and must not manufacture a pair. One row per deliverable §8's column names; 04 and the README
# carry no closure section because no row names them, and a pair routed to either stops the run
# at the map check below. Each row is: <deliverable> <file>|<heading marker>.
cat > "$w/map" <<'MAP'
01 01-architecture-overview.md|finding-register row this deliverable discharges
02 02-dependency-inventory.md|row this document discharges
03 03-modernization-roadmap.md|Where the security register attaches
05 05-aspnet-core-migration-approach.md|findings of deliverable 09 this document closes
06 06-azure-hosting-recommendations.md|findings this document closes
07 07-effort-risks-sequencing.md|Where the security register attaches
08 08-technical-debt-register.md|Ownership and cross-reference map
10 10-build-and-deployment-requirements.md|finding-register rows this deliverable discharges
11 11-cloud-readiness-assessment.md|finding-register rows this deliverable discharges
12 12-migration-blockers.md|rows this document discharges
MAP

# The section runs from its own heading to the next heading of the same or a higher level. Fenced
# blocks are tracked, because a shell comment inside a fence begins with '#' and would otherwise
# read as a heading and truncate the section -- 05's own extraction fence sits inside its closure
# section, ahead of the table. Exits 9 unless the marker matched exactly one heading.
section() {
  awk -v m="$2" '
    /^[[:space:]]*```/ { fenced = !fenced; if (inside) print; next }
    !fenced && /^#+[ \t]/ {
      lvl = 0; while (substr($0, lvl + 1, 1) == "#") lvl++
      if (inside && lvl <= start) inside = 0
      if (!inside && index($0, m) > 0) { inside = 1; start = lvl; found++; next }
    }
    inside { print }
    END { exit (found == 1 ? 0 : 9) }
  ' "$1"
}

: > "$w/actual"; : > "$w/attrib"
while IFS='|' read -r left marker; do
  d=${left%% *}; f=${left#* }
  [ -r "$f" ] || die "consumer $d: $f cannot be read"
  section "$f" "$marker" > "$w/section" ||
    die "consumer $d: no single heading in $f contains '$marker' -- its designated closure section could not be located, and this block does not fall back to a whole-file scan"
  ids=$(grep -o 'F-09-[0-9][0-9]' "$w/section" | sort -u)
  [ -n "$ids" ] || die "consumer $d: its designated closure section names no F-09 identifier"
  printf '%s\n' "$ids" | sed "s/\$/ $d/" >> "$w/actual"
  # Rule 2, applied inside the same section: identifiers the consumer labels as attributions
  # rather than closures, read out of its own labelled paragraph and not from a list kept here.
  awk '/none of them is a closure/,/^$/' "$w/section" |
    grep -o 'F-09-[0-9][0-9]' | sort -u | sed "s/\$/ $d/" >> "$w/attrib"
done < "$w/map"
sort -u "$w/attrib" -o "$w/attrib"; sort -u "$w/actual" -o "$w/actual"
comm -23 "$w/actual" "$w/attrib" > "$w/closures"

# Every consumer §8 names must be one this block actually reads, or the run reports on a corpus it
# has not read. This fails closed instead of printing the pair as an ordinary MISSING.
cut -d" " -f2 "$w/expected" | sort -u > "$w/named"
cut -d" " -f1 "$w/map"      | sort -u > "$w/mapped"
comm -23 "$w/named" "$w/mapped" > "$w/unmapped"
[ ! -s "$w/unmapped" ] ||
  die "§8 assigns a pair to $(tr "\n" " " < "$w/unmapped")-- no designated closure section is mapped for it"

# --- 3. The exclusion is validated, not trusted, and it fails closed --------------------------
comm -12 "$w/attrib" "$w/expected" > "$w/invalid"
echo "attributions excluded (identifier, deliverable):"; sed 's/^/  /' "$w/attrib"
sed 's/^/INVALID EXCLUSION: /' "$w/invalid"

# --- 4. Set comparison: missing, extra, misplaced; pass only on exact equality -----------------
comm -23 "$w/expected" "$w/closures" > "$w/missing"
comm -13 "$w/expected" "$w/closures" > "$w/extra"
printf 'expected pairs = %s\nactual pairs   = %s\n' "$(wc -l < "$w/expected")" "$(wc -l < "$w/closures")"
sed 's/^/MISSING: /' "$w/missing"; sed 's/^/EXTRA:   /' "$w/extra"
join <(cut -d" " -f1 "$w/missing" | sort -u) <(cut -d" " -f1 "$w/extra" | sort -u) |
  sed 's/^/MISPLACED: /'
if [ ! -s "$w/missing" ] && [ ! -s "$w/extra" ] && [ ! -s "$w/invalid" ]
then echo "PASS -- the two pair sets are identical"; exit 0
else echo "FAIL -- see the lines above"; exit 1
fi
```

**Verbatim output, exit code 0, run from the repository root.** `09` itself is not in the map, because
it is the source rather than a consumer, and neither are `04` and the `README`, because no row names
either of them:

```text
attributions excluded (identifier, deliverable):
  F-09-03 12
  F-09-05 12
  F-09-08 12
  F-09-34 12
expected pairs = 52
actual pairs   = 52
PASS -- the two pair sets are identical
```

**Both failure modes were provoked rather than described, on a scratch copy of the thirteen files that
left the repository untouched, and each provocation's exit status was read rather than assumed.**
Moving `F-09-30`'s designated closure out of 11's section and into 10's — the compensated swap a count
comparison cannot see, since both totals stay at 52 — prints, in place of the pass line, and exits
**1**:

```text
MISSING: F-09-30 11
EXTRA:   F-09-30 10
MISPLACED: F-09-30
FAIL -- see the lines above
```

And making §8 assign F-09-03 to `12`, which would turn one of 12's four attributions into a real
closure, prints `INVALID EXCLUSION: F-09-03 12` above `MISSING: F-09-03 12`, fails, and exits **1** —
which is the fail-closed property the exclusion rests on, demonstrated rather than claimed. A third
provocation exercises the scoping itself rather than a failure: an `F-09-24` mention added to 11's
ordinary prose, outside its §7.1 closure section, leaves the run at `expected pairs = 52`,
`actual pairs = 52`, `PASS` and exit **0**, because a passing mention is not a discharge. The
whole-file form this block previously used reported `EXTRA: F-09-24 11` there — a confident failure
against a correct corpus, which is the second reason the extraction is scoped. Two internal-error
paths were provoked on the same scratch copy and both exited **3** with a named message and no
verdict line: renaming 11's closure heading, which reports that the section could not be located
rather than falling back to a whole-file scan, and making §8's register rows unparseable, which
reports an empty expected set rather than passing on a comparison of nothing against nothing.

---

*Deliverable 09 of 13. Consumes 01 (architecture) and 02 (dependencies); feeds 12 (migration
blockers) alongside 10 and 11. Every finding cites a repository path and locator and names the
editions it holds in; every count and absence carries the command that reproduces it. **No tracked
repository file was changed** — the defects recorded here are documented for approval, and remediation
begins only once the assessment and modernization plan are approved. §9.1 accounts for the untracked
build output the assessment's own restores left in the checkout, and for its removal.*
