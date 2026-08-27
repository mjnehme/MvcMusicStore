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

It consumes deliverable 01 (architecture) and deliverable 02 (dependencies), and it feeds deliverable
12 (migration blockers) alongside 10 and 11.

### 1.2 What this document is not

It is not a remediation plan, a penetration test or a threat model. It does not assert a CVE
identifier of its own: the only advisory identifiers anywhere in this assessment are the ones NuGet's
restore audit emitted verbatim, and deliverable 02 owns them (§8.2 there). It does not rank the
editions against each other as products — MVC 4 and MVC 3 are retained read-only as historical
references and as the behavioural baseline, and neither is a migration target.

It also does not describe the target-state security design. That belongs to deliverables 05 and 06,
and section 1.5 draws the line.

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
administrator credential is quoted verbatim in this document, with its file and line.** It is not
redacted, masked or paraphrased. It is already public — in three tracked files and in the
repository's own README — and the finding *is* that it is committed in cleartext. A redacted finding
is an unverifiable finding, and a reader who cannot confirm the value cannot confirm the severity.
Rotating or obscuring it would also be a repository change, which section 1.3 forbids.

### 1.4 Authoring contract, and the absence of user rules

`review_rules` returns exactly **"No user rules provided."** No project rule constrains this
document, no rule forces any file into its scope, and none is invented in its place. Their absence is
not licence to lower the bar; the assessment's own contracts bind instead, and five govern this file:

1. **Every claim about the existing system carries an inline `[<path>:<locator>]` citation** at the
   point the claim is made, repository-relative and resolving in the checkout. There is no trailing
   reference list — a citation collected at the end cannot be checked against the sentence it
   supports.
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
   Section 4.9 is the case in point: the disclosure mechanics of one line of MVC 4 were settled by
   running the shipped assembly, and the result corrected the characterization this assessment
   started from.

### 1.5 What this document does not own

One fact, one owner. This document owns **present posture and present risk**. It does not decide the
target state, and where a finding has a forward decision attached, the decision belongs to the
deliverable named below and is cross-referenced rather than restated — a restatement in different
words reads downstream as a second decision.

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

- **MVC 5** has the most complete anti-forgery coverage and the only anti-forgery-protected sign-out,
  and it also ships a plaintext administrator credential and provisions it from an `async void`
  method whose failures nothing can observe.
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
| Lockout | **not determinable** | **none in the code path** | **none — absent from the schema** | §5.3, §4.3, §3.3 |
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
| Credential store committed to source control | **yes** | **yes** | **yes** | §6.9 |
| Live `customErrors` element | no | no | no | §6.10 |
| Scaffolded-but-disabled external login | none | yes — 6 packages | yes — 4 packages | §6.11 |

### 2.3 The distribution is the finding

Read as a whole, the table says something more useful than "a 2013 tutorial application has security
problems". This codebase is **careful in specific, checkable places** — all eight of MVC 5's account
POST actions validate their token [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:55], [:88],
[:113], [:147], [:199], [:236], [:264], [:301]; the checkout confirmation page verifies that the
signed-in user owns the order it is about to display
[src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:71-73]; the post-sign-in redirect is
guarded against open redirection by `Url.IsLocalUrl`
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:384]; three of the four administration
`Albums.Find` calls guard their result and return `HttpNotFound()`
[src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:33-35], [:74-76], [:106-108]; and the
cart removal query is correctly scoped to the caller's own cart
[src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:64-66]. Section 7 collects these.

It is **alarming in equally specific places**: a credential in cleartext in two editions, a mutating
`GET` in all three, five unprotected state-changing POSTs in each of the two newer editions, no audit
trail anywhere, and every edition's credential database committed to source control with its password
material intact.

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
by assembly attribute [src/MVC5/MvcMusicStore/Startup.cs:1] and calls `ConfigureAuth` then
`ConfigureApp` [src/MVC5/MvcMusicStore/Startup.cs:11-13].

The credential store is ASP.NET Identity **1.0**, EF-mapped and owned by the application:
`ApplicationDbContext : IdentityDbContext<ApplicationUser>` [src/MVC5/MvcMusicStore/Models/IdentityModels.cs:10]
bound to `DefaultConnection` [src/MVC5/MvcMusicStore/Models/IdentityModels.cs:13]. Deliverable 01 §8.1
owns the architecture; the security consequence of the *version* is §3.3 below, because Identity 1.0
is the generation that predates lockout.

### 3.2 Authorization posture, and its enforcement points

**Editions: MVC 5** (MVC 4 is identical in shape — §4.2; MVC 3 differs materially — §5.2).

Authorization is entirely declarative, and there are five enforcement points, all of them
controller-level:

| Enforcement point | Effect |
| --- | --- |
| `[Authorize]` on `AccountController` [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:15] | Authenticated by default, with `[AllowAnonymous]` opening the sign-in, registration and external-callback actions individually — eight occurrences at [:44], [:54], [:78], [:87], [:198], [:208], [:263], [:310] |
| `[Authorize]` on `CheckoutController` [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:8] | The whole checkout requires authentication |
| `[Authorize(Roles = "Administrator")]` on `StoreManagerController` [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:12] | The entire administration surface requires the role |
| Ownership check inside `Checkout.Complete` [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:71-73] | The order confirmation is shown only if `o.Username == User.Identity.Name` |
| Cart scoping inside `ShoppingCart.RemoveFromCart` [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:64-66] | The removal targets only a row whose `CartId` matches the caller's cart key |

`StoreController`, `HomeController` and `ShoppingCartController` carry no authorization attribute at
all, which is correct for a public catalogue and an anonymous cart. The two data-scoping checks in the
last two rows are genuine controls and are credited in section 7 — but note the asymmetry recorded in
§6.2: the removal *mutation* is scoped, and the album-title *read* that accompanies it is not.

**Finding F-09-01 — the declarative model has no defence in depth.** Severity **Low** on its own.
Every one of the five points above is the *only* thing standing between an anonymous request and the
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
sign-in cookie [:20]. Nothing else is set, anywhere. `git grep -in 'machineKey|sessionState|requireSSL|httpCookies' -- '*.config'` returns **0**
(§9).

The policy is therefore not *absent*. It exists, it is in force on every request, and it is whatever
the framework's defaults happen to be:

| Policy | Where it comes from | Stated in the repository? |
| --- | --- | --- |
| Password minimum length and complexity | `UserManager` defaults; no `PasswordValidator` is assigned anywhere in the edition | **no** |
| Lockout threshold and duration | **no lockout exists** — see the finding below | **no** |
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

**Finding F-09-03 — there is no account lockout, and the schema cannot express one.** Severity
**High**. Sign-in is a single call, `UserManager.FindAsync(model.UserName, model.Password)`
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:60], with no attempt counter, no delay and
no lockout on failure; the failure path adds a message and redisplays the form
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:68], [:73]. That is not an omission in the
application — ASP.NET Identity **1.0** predates lockout entirely. A printable-string inspection of the
shipped credential database finds `PasswordHash` and `SecurityStamp` but **no** occurrence of
`LockoutEnabled`, `LockoutEndDateUtc`, `AccessFailedCount`, `TwoFactorEnabled` or `EmailConfirmed`
[src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205.mdf], which is consistent with
the 1.0 schema and with the pinned package version `1.0.0` that deliverable 02 records (§3.1 there).
The probe command is in §9, and its epistemic limit is stated honestly: **string-probing a binary is
evidence, not proof** — it cannot distinguish an absent column from one stored in a form the probe
does not surface. Deliverable 05 must treat an authoritative `sys.columns` query as the gate on the
Identity migration; the security conclusion drawn here is narrower and safe either way, because the
*code path* contains no lockout regardless of what the schema holds. Combined with the absence of
transport protection (§6.5) and of any logging that would reveal an attack in progress (§6.8), an
online password-guessing attack against this edition is unthrottled and unrecorded.

### 3.4 Identity and account flows

**Editions: MVC 5.**

Fifteen public actions make up the account surface [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:45-317].
The security-relevant properties:

- **Sign-in does not enumerate users.** The failure message is identical for an unknown user and a
  wrong password [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:68]. Credited in §7.
- **Sign-out is a token-protected `POST`.** The action carries both `[HttpPost]` and
  `[ValidateAntiForgeryToken]` [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:300-301], and
  its call site is a real form with a token, submitted by script
  [src/MVC5/MvcMusicStore/Views/Shared/_LoginPartial.cshtml:4-6], [:12]. This is the correct design
  and MVC 3 does not have it (§5.4).
- **The external-login flow carries its own anti-forgery value.** A dedicated key
  [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:336] is passed to
  `GetExternalLoginInfoAsync` when linking a login to an existing account
  [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:247]. Note the asymmetry: the *callback*
  after an initial external sign-in does not pass it
  [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:211], [:275] — which is the framework's own
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
saves it [:37] and overwrites the session cart key [:39]. There is no transaction spanning the
identity write and the catalogue write, and none is possible — the cookie is issued into the HTTP
response, outside any database transaction. If the cart migration throws, the failure is swallowed by
nothing and logged by nothing (§6.8), and whether the already-issued cookie reaches the browser is
not determined by anything in the repository. The security-relevant half of this is the ordering: the
cart is reassigned to a user whose sign-in has not been confirmed complete. Deliverable 05 owns the
target ordering and the idempotence requirement; the finding here is that today the ordering is the
riskier one and nothing observes the failure.

### 3.5 Secret handling — the plaintext administrator credential

**Editions: MVC 5 and MVC 4** (identically; MVC 3 has no provisioning at all — §5.5).

**Finding F-09-05 — a working administrator username and password are committed to source control in
cleartext, and the repository's own documentation republishes them.** Severity **Critical**.

The credential is two `appSettings` entries:

```xml
<add key="DefaultAdminUsername" value="Administrator"/>
<add key="DefaultAdminPassword" value="YouShouldChangeThisPassword"/>
```

at [src/MVC5/MvcMusicStore/Web.config:16-17], and **byte-identically** at
[src/MVC4/MvcMusicStore/Web.config:25-26]. They are read at application startup by both editions —
MVC 5 at [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:23-24], MVC 4 at
[src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:23-24] — and used to create a real account holding a
real role. This is not a template placeholder that a deployment overrides; nothing in either edition
requires it to be overridden, and the account is created on first run either way.

The value is also published in prose, under a heading that describes exactly what it is:
[src/MVC5/README.md:75-85], with the username and password on [:78-79] and the `Web.config` snippet
reproduced at [:83-84]. So the credential is discoverable without reading any code.

Three aggravating facts, each cited independently:

1. **The account is a full administrator.** The role it receives is `Administrator`
   [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:25], which is the exact string guarding the entire
   administration surface [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:12].
2. **The credential is not the only committed secret material — the resulting password hash is
   committed too.** The Identity database that stores it is tracked
   [src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205.mdf], and a string probe finds
   both `PasswordHash` and the literal `Administrator` inside it (§6.9, §9).
3. **Rotation does not remediate it.** Changing the value in `Web.config` leaves every prior value in
   git history and leaves the committed database's hash intact. This is why deliverable 08 records
   the committed binaries as debt requiring either history rewriting or explicit acceptance, and why
   the remediation here is not "change the password" but "remove the mechanism".

Deliverable 05 owns the replacement — provisioning moves out of application startup to an operator
command, and the credential leaves configuration entirely. **Nothing is changed here.**

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
`AddToRoleAsync` at [:43] — which §3.6's first finding makes a silent and therefore plausible
outcome — every subsequent startup finds the user present, skips the branch entirely, and never adds
the membership. The account exists, cannot administer anything, and no run will ever fix it.

MVC 4 does not have this defect (§4.6), which makes the pair a concrete, in-repository demonstration
of why idempotence must be checked **per operation** rather than overall. Deliverable 05 owns that
requirement for the replacement command; this document supplies the evidence for it.

### 3.7 Anti-forgery — emission is broad, validation is not

**Editions: MVC 5** (MVC 4 has the same *shape* with different counts — §4.7; MVC 3 has neither —
§5.6).

`@Html.AntiForgeryToken()` appears **10** times across MVC 5's views, in 10 distinct files
(`git grep -o 'Html.AntiForgeryToken' -- 'src/MVC5/*.cshtml' | wc -l` → `10`; §9). Emission is broad.

**Finding F-09-08 — five of thirteen state-changing POST actions do not validate the token, and two
of the five render one on the page that nothing checks.** Severity **High**.

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
| `StoreManager.Create` | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:53-54] | Inserts an album [:58-59] | **yes** [src/MVC5/MvcMusicStore/Views/StoreManager/Create.cshtml:11] |
| `StoreManager.Edit` | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:86-87] | Full-entity update [:91-92] | **yes** [src/MVC5/MvcMusicStore/Views/StoreManager/Edit.cshtml:11] |
| `StoreManager.DeleteConfirmed` | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:116-117] | Deletes an album [:119-121] | no |
| `ShoppingCart.RemoveFromCart` | [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:54-55] | Decrements or removes a cart row [:65], [:67] | no — posted by script without a form [src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:17] |
| `Checkout.AddressAndPayment` | [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:25-26] | Writes the order and its details [:44], [:48], [:51] | no |

**The first two rows are the finding in its sharpest form: the token is emitted into the form and the
action does not look at it.** A reader auditing the views would conclude the administration surface is
protected. It is not. Emission is not a control; validation is.

**Methodology note, because a plausible command undercounts this.** `grep -c '\[HttpPost\]'` reports
**2** POST actions in `StoreManagerController`, not 3 — its third is declared
`[HttpPost, ActionName("Delete")]` on a single line
[src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:116], which the bracketed literal does
not match. The correct count comes from `grep -n 'HttpPost'` (§9). Any audit of this repository that
uses the literal form will miss the album-delete action, which is the most destructive of the three.

Deliverable 05 owns the target policy, the conversion of the AJAX endpoint's token transport and the
verb change discussed in §6.1. This document records only the present coverage.

### 3.8 Session and cart identity

**Editions: MVC 5, MVC 4 and MVC 3** — the mechanism is identical in all three; see §6.2 for the
cross-edition finding and its per-edition locators.

MVC 5's cart key is held in session under a constant
[src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:19] and resolved by `GetCartId`
[src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:161-180]: a signed-in user's key is their **login
name** [:167], an anonymous visitor's is a fresh `Guid.NewGuid()` [:172]. No `<sessionState>` element
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

- **The error view discloses nothing.** `src/MVC5/MvcMusicStore/Views/Shared/Error.cshtml` is nine
  lines. It declares `@model System.Web.Mvc.HandleErrorInfo` [:1] but renders **no property of that
  model** — only two static headings [:7-8]. There is no exception message, no stack trace and no
  request detail in it. **Naming the type is not disclosing it**, and the reason this view must be
  rewritten during the port is that `HandleErrorInfo` does not exist in ASP.NET Core, which is
  deliverable 12's territory and not a security finding.
- **`HandleErrorAttribute` only engages when custom errors are enabled, and no edition enables
  them.** No live `<customErrors>` element exists anywhere: all **24** occurrences of the element sit
  inside commented example blocks in the six XDT transform files, and **zero** appear in any of the
  three live `Web.config` files (§6.10, §9). With the element undeclared, ASP.NET's default is
  `RemoteOnly` — so a request originating on the server host itself receives the detailed ASP.NET
  error page rather than `Error.cshtml`. Combined with `<compilation debug="true" …/>`
  [src/MVC5/MvcMusicStore/Web.config:33], which the Release transform removes
  [src/MVC5/MvcMusicStore/Web.Release.config:18] but which is what the committed configuration
  carries, the shipped posture leaves error-detail behaviour to a framework default nobody stated.

**Finding F-09-10 — the checkout swallows every exception silently.** Severity **Medium**. The entire
order-writing transaction is wrapped in a bare `catch` with no exception variable
[src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:58-61]. The exception is discarded, the
view is redisplayed with no message, and — with no logger anywhere (§6.8) — nothing records that a
customer's order failed. This is the inverse of a disclosure problem and is just as serious: the
failure is undetectable and unattributable. It holds in **all three editions**; see §6.8 for the
per-edition locators.

### 3.11 PII, retention and encryption

**Editions: MVC 5, MVC 4 and MVC 3** — the entity is materially the same in all three; see §6.8.

MVC 5's `Order` entity carries nine personal-data fields: `FirstName`
[src/MVC5/MvcMusicStore/Models/Order.cs:23], `LastName` [:28], `Address` [:32], `City` [:36], `State`
[:40], `PostalCode` [:45], `Country` [:49], `Phone` [:54] and `Email` [:61], alongside `Username`
[:18] which links the record to an identity. They are persisted as ordinary columns by
`storeDB.SaveChanges()` [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:51]. No encryption,
no hashing, no tokenization and no column-level protection appears anywhere. §6.8 records the absence
of retention and audit controls with its commands.

One point in the application's favour, because it bounds the exposure: **no payment data is
collected or stored.** The `Order` entity has no card, account or token field, and the checkout's
only gate is a promo-code string comparison
[src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:33-34] against a compile-time constant
[:12]. There is no payment integration to assess — a fact deliverable 01 records as an out-of-scope
capability and which materially reduces the regulatory surface here.

### 3.12 Data protection

**Editions: MVC 5, MVC 4 and MVC 3.** See §6.6 — no edition declares `<machineKey>` or any key
material, and the consequence for cookie and token integrity across instances is recorded there.

### 3.13 Auditability

**Editions: MVC 5, MVC 4 and MVC 3.** See §6.8 — there is no logging construct of any kind anywhere
in the repository. For MVC 5 specifically, the combination that matters is F-09-06 (silent
provisioning failure), F-09-10 (silent checkout failure) and the total absence of a log: three
distinct security-relevant events, none of which leaves a trace.

### 3.14 Dependency exposure

**Editions: MVC 5.**

Deliverable 02 owns the inventory and the advisory evidence; two of its findings are security
conclusions and are cross-referenced rather than re-derived.

- **The one authentication middleware actually enabled is named by the advisory audit.** Deliverable
  02 §8.2 records that NuGet's own restore-time audit names **14 of the 63 pins** across 43
  `NU1902`/`NU1903` warnings, 9 of them high severity, and that MVC 5's named pins include
  `Microsoft.Owin` `2.0.0`, `Microsoft.Owin.Security.Cookies` `2.0.0` and
  `Microsoft.AspNet.Identity.Owin` `1.0.0` — that is, the cookie authentication middleware registered
  at [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:14-18] and the Identity/OWIN bridge behind the
  sign-in path. Severity is deliverable 07's to set; the security observation is that the advisories
  land on the authentication path rather than on a peripheral package.
- **Client-side libraries are self-hosted with no integrity or policy control.** Deliverable 02 §8.1
  records that every client library is served from the application's own `Scripts/` and `Content/`
  directories rather than a versioned CDN, and that none is served with a Subresource Integrity
  attribute or under a Content Security Policy — neither is declared anywhere in the repository
  (§6.5 records the header absence with its command).
- **Nothing in the repository retains or gates on that audit output** (deliverable 02 §8.2, F-02-23):
  43 warnings scroll past on every clean restore, there is no dependency-scanning configuration
  (`git ls-files | grep -E '^\.github/|dependabot|renovate' | wc -l` → `0`; §9), and no artifact
  keeps them. Deliverable 03 owns the gate.

---

## 4. MVC 4 — SimpleMembership with Forms authentication

*Every finding in this section holds in **MVC 4** unless its Editions line says otherwise. MVC 4 is
not a migration source, but it is the behavioural baseline for the shared surfaces and it carries two
findings the other editions do not.*

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

Identical in shape to MVC 5 (§3.2), with the same three controller-level attributes —
`[Authorize]` on the account controller [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:16],
`[Authorize]` on checkout [src/MVC4/MvcMusicStore/Controllers/CheckoutController.cs:8] and
`[Authorize(Roles = "Administrator")]` on the administration surface
[src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:12] — and the same two in-action
data-scoping checks: the order-ownership check on the confirmation page
[src/MVC4/MvcMusicStore/Controllers/CheckoutController.cs:71-73] and the cart-scoped removal
[src/MVC4/MvcMusicStore/Models/ShoppingCart.cs:64-66]. F-09-01 applies unchanged.

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
  delay; the failure path adds a generic message [:47]. F-09-03's *conclusion* — unthrottled,
  unrecorded online password guessing — holds here too, for a different reason: not a schema that
  cannot express lockout, but a code path that never consults one.
- **No `<sessionState>` and no `<machineKey>`** — §6.6, with its command.

### 4.4 Identity and account flows

**Editions: MVC 4.**

Fourteen public actions [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:24-330]. The
security-relevant properties, and MVC 4 does well on three of them:

- **Sign-out is a token-protected `POST`** [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:66-67],
  posted from a real form with a token [src/MVC4/MvcMusicStore/Views/Shared/_LoginPartial.cshtml:5].
  MVC 3 does not have this (§5.4).
- **Sign-in does not enumerate users** — one message for both failure modes
  [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:47].
- **A provider exception is mapped to a safe message rather than surfaced.**
  `catch (MembershipCreateUserException e)` [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:105]
  passes only `e.StatusCode` through a lookup [:107]. This is the correct handling — and it makes the
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

The account created is `Administrator` holding the role `Administrator`
[src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:25], which is the exact string guarding the
administration surface [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:12].

### 4.6 MVC 4's provisioning is more idempotent than MVC 5's

**Editions: MVC 4 (favourably) contrasted with MVC 5.**

This is the one place the older edition is unambiguously better, and it is worth recording precisely
because the target design depends on the distinction.

The three steps in the table above are **three independent checks**
[src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:29], [:32], [:35]. Each guards its own operation. A
prior run that created the user but failed before granting the role is therefore **repaired** on the
next startup: the user check short-circuits, the role check short-circuits, and the membership check
finds the gap and closes it at [:36].

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

The seven protected are all in the account controller
[src/MVC4/MvcMusicStore/Controllers/AccountController.cs:35], [:67], [:89], [:119], [:163], [:227],
[:271]. The five unprotected are the same five surfaces as MVC 5's:

| Unprotected POST | Declaration | What it changes |
| --- | --- | --- |
| `StoreManager.Create` | [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:53-54] | Inserts an album |
| `StoreManager.Edit` | [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:86-87] | Full-entity update [:91] |
| `StoreManager.DeleteConfirmed` | [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:116-117] | Deletes an album |
| `ShoppingCart.RemoveFromCart` | [src/MVC4/MvcMusicStore/Controllers/ShoppingCartController.cs:54-55] | Decrements or removes a cart row |
| `Checkout.AddressAndPayment` | [src/MVC4/MvcMusicStore/Controllers/CheckoutController.cs:25-26] | Writes the order and its details |

**Finding F-09-12 — one edition difference cuts in MVC 4's favour and is easy to invert.** Severity
informational, recorded to prevent a wrong conclusion. All eight of MVC 4's emitted tokens are in
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
[:16]; `MusicStoreEntities` targets `(LocalDB)\v11.0`
[src/MVC4/MvcMusicStore/Web.config:19] with `Integrated Security=True` [:21] and attaches its own
[:20]. Neither carries a stored password, which is the same point in its favour recorded in §3.9.

What makes MVC 4 worse than MVC 5 is that the privilege requirement is not implied — it is exercised
explicitly, in application code, on the credential store:

- `((IObjectContextAdapter)context).ObjectContext.CreateDatabase()`
  [src/MVC4/MvcMusicStore/Filters/InitializeSimpleMembershipAttribute.cs:37] — the application
  **creates the credential database** if it does not exist [:34].
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

**Where it is not.** `src/MVC4/MvcMusicStore/Views/Shared/Error.cshtml` is **11 lines** and contains
only generic apology text: a title assignment [:1-3], a heading [:5] and a paragraph offering to go
back [:7-11]. It declares **no `@model`**, reads **no exception**, and renders **no** message, type,
stack trace or request detail. It is not the disclosure site, and any finding that places it there is
wrong. (MVC 5's error view is also innocent, for a slightly different reason — §3.10.)

**Where it is.** In the POST `Manage` action
[src/MVC4/MvcMusicStore/Controllers/AccountController.cs:162-164], in the branch taken when the
signed-in account has no local password [:194-195]:

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
[:190]. One action, two branches, two opposite handling decisions.

The action then redisplays the view [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:219];
`Manage.cshtml` selects the set-password partial in exactly this branch
[src/MVC4/MvcMusicStore/Views/Account/Manage.cshtml:14-21]; and that partial renders
`@Html.ValidationSummary()` [src/MVC4/MvcMusicStore/Views/Account/_SetPasswordPartial.cshtml:10]. So
the exception object is placed on a channel that is rendered into the response.

**What actually reaches the browser — tested, not assumed.** Rather than assert the consequence, this
assessment ran it. A harness compiled against **MVC 4's own committed
`System.Web.Mvc.dll` 4.0.20710.0** — the exact assembly this edition binds to, taken from
`src/MVC4/MvcMusicStore/packages/Microsoft.AspNet.Mvc.4.0.20710.0/lib/net40/` — constructed the same
model-state entry and rendered a real `Html.ValidationSummary`. The command is in §9. Results:

| Probe | Result |
| --- | --- |
| `AddModelError("", exception)` → `ModelError.ErrorMessage` | `""` (length 0); the `Exception` is retained on the error |
| `Html.ValidationSummary()` output | `<div class="validation-summary-errors"><ul><li style="display:none"></li></ul></div>` |
| `Html.ValidationSummary(true)` output | identical |
| Exception text present in either output | **no** |
| Control: `AddModelError("", "text")` → summary output | `<li>text</li>` — the message **is** rendered, confirming the harness exercises the helper |

**So the honest finding is two findings, and neither is "the stack trace is shown to the user".** The
framework's summary helper renders `ModelError.ErrorMessage`, which the exception overload leaves
empty, so the shipped view emits an error box containing an empty, hidden list item.

**Finding F-09-14 — a raw exception is placed on a response-bound channel, where a change in
rendering turns it into disclosure with no change at the call site.** Severity **Medium**. The
exception object *is* in model state and *is* carried into view rendering. Any of the following turns
that into real disclosure without touching :213: a custom or third-party validation summary that
reads `ModelError.Exception`; a helper or filter that serializes model state to JSON for an API or
AJAX consumer; a change in the framework's message-resolution behaviour across a major version; or a
port to a framework whose model-error rendering differs. A port is precisely what deliverable 05
plans, which is why this is recorded as a live hazard rather than a historical curiosity.

**Finding F-09-15 — as shipped, the same line destroys the exception entirely, which is the more
immediate defect.** Severity **Medium**. The user sees an error container with no message and no
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
  [src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs:17-29]). §6.11 records the general finding; the
  MVC 4 instance is the largest, because an entire OAuth and OpenID relying-party stack is deployed
  for a disabled feature.
- **MVC 4's named advisory pins are client-side and serialization packages** — deliverable 02 §8.2
  records 14 warnings against `jQuery` `1.7.1.1`, `jQuery.UI.Combined` `1.8.20.1`,
  `jQuery.Validation` `1.9.0.1` and `Newtonsoft.Json` `4.5.6`, two of them high severity. All are
  self-hosted with no Subresource Integrity attribute and under no Content Security Policy (§6.5).
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
the oldest and least complete edition, and it is the one whose posture the repository **cannot fully
determine** — §5.3 is the boundary of what can be asserted from the checkout.*

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

**Finding F-09-17 — every account MVC 3 creates is given the literal password-recovery question
`"question"` and the literal answer `"answer"`.** Severity **High**, conditional on the host's
provider settings — and the condition is worth spelling out rather than hiding behind.

Registration calls
`Membership.CreateUser(model.UserName, model.Password, model.Email, "question", "answer", true, null, out createStatus)`
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:94]. The fourth and
fifth arguments are the recovery question and answer, and they are **compile-time string literals** —
identical for every account the application will ever create. The seventh argument, `true`, is
`isApproved`, so accounts are auto-approved with no confirmation step.

If the machine-level provider has `requiresQuestionAndAnswer` enabled together with
`enablePasswordReset` or `enablePasswordRetrieval` — the classic provider's own defaults enable reset
and question-and-answer — then the recovery answer for **every** account in the system is the
five-character string `answer`, and account takeover requires only a user name. That is why the
severity is High: the precondition is the provider's *default* posture, not an unusual one. It is
recorded as conditional because, per F-09-16, this repository cannot settle what the host has
configured. It is the single strongest reason the host verification deliverable 10 owns must actually
be performed rather than assumed.

MVC 4 and MVC 5 are not exposed: neither uses a question-and-answer recovery mechanism at all.

### 5.2 Authorization posture — a guarded surface behind a role nothing creates

**Editions: MVC 3.**

**Finding F-09-18 — MVC 3's administration surface is guarded by the `Administrator` role, and no
code in the edition ever creates that role or grants it to anyone.** Severity **High** as an
availability finding, and it is genuinely security-relevant in both directions.

The guard is real: `[Authorize(Roles = "Administrator")]` on the whole controller
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:12]. The
provisioning is absent: MVC 3 has **no `App_Start` folder at all**
(`test -d src/MVC3/MvcMusicStore-Completed/MvcMusicStore/App_Start` → false; §9), and its
`Application_Start` runs only four registrations — the EF initializer
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Global.asax.cs:34], area registration [:36], the
global filter [:38] and routes [:39]. There is no `CreateAdminUser`, no `Roles.CreateRole` call and no
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
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:116], [:125]. MVC 4
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
— the comment directly above it says so explicitly, `// GET: /Account/LogOff` [:67] — and it performs
a state change, `FormsAuthentication.SignOut()` [:71], before redirecting [:73]. Any third-party page
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
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:43], [:45] — the
  safer ordering, and the exact inverse of MVC 5's, which is F-09-04. The same ordering holds on its
  registration path [:98], [:100].
- **Its failure messages are generic and its exceptions are discarded rather than surfaced.** Sign-in
  failure [:58], registration failure through a status-code lookup [:105], and change-password
  failure through `catch (Exception)` with no variable and a generic message [:140-143], [:151]. MVC 3
  has no equivalent of MVC 4's F-09-14.

### 5.5 Secret handling

**Editions: MVC 3.**

No credential of any kind appears in MVC 3's configuration: its `appSettings` block holds three
framework settings and nothing else
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:8-12], and its single connection string
carries no password [:55-59]. F-09-05 does **not** apply to MVC 3, and this assessment does not
extend it there.

That is a consequence of having no provisioning rather than a decision, and it comes at the cost of
F-09-18. But two hard-coded secret-adjacent literals do exist in the edition: the recovery question
and answer of F-09-17
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:94]. And MVC 3's
credential store is committed to the repository like the other two — as a tutorial asset,
`src/MVC3/MvcMusicStore-Assets/Data/ASPNETDB.MDF`, with `aspnet_Membership`, `aspnet_Users`,
`Password` and `PasswordSalt` all present in it (§6.9, §9).

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
| MVC 3 | `[Bind(Exclude = "OrderId")]` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/Order.cs:8] | **Everything binds except `OrderId`** — including `OrderDate` [:15], `Username` [:18] and `Total` [:63] |
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
   followed by `storeDB.SaveChanges()` [:156].
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
- **MVC 3 needs two different engines at once, and the credential one is the undeclared one.** The
  catalogue is SQL Server Compact; the credential store is a SQL Server instance reached through the
  machine-level provider (F-09-16). Deliverable 10 owns the prerequisite; the security point is that
  the store holding password material is the one the repository says least about.

The same destructive initializer as the other two editions is registered
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Global.asax.cs:34], against
`DropCreateDatabaseIfModelChanges<MusicStoreEntities>`
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/SampleData.cs:9] — §6.7.

### 5.9 Error disclosure, PII, data protection and auditability

**Editions: MVC 3.**

- **Error disclosure: none, and the error view is innocent.** The global filter is registered inline
  in `Global.asax.cs` rather than an `App_Start` file
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Global.asax.cs:17], and
  `src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Views/Shared/Error.cshtml` is nine lines of generic
  apology with no `@model` and no exception rendering. MVC 3 has no equivalent of F-09-14. The
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
  payload is encoding for the wrong context. Recorded as **Low**, F-09-23, for both directions.

### 5.10 Dependency exposure

**Editions: MVC 3.**

Deliverable 02 owns the inventory. Two security conclusions:

- **MVC 3's MVC framework assembly is not a package.** `System.Web.Mvc, Version=3.0.0.0` is
  referenced with no `HintPath`
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:42], so it resolves from a
  machine-wide install of the out-of-support ASP.NET MVC 3 Tools Update (deliverable 02 F-02-11). The
  security consequence: the framework version actually serving requests is a property of the host and
  is not pinned by this repository, and the product it comes from receives no security servicing.
- **Its three named advisory pins are the oldest in the repository** — deliverable 02 §8.2 records 14
  warnings against `jQuery` `1.5.1`, `jQuery.UI.Combined` `1.8.11` and `jQuery.Validation` `1.8.0`.
  Self-hosted, no Subresource Integrity, no Content Security Policy (§6.5).
- **MVC 3 has no external-login surface at all**, so §6.11 does not apply to it — deliverable 01 §8.3
  records the same, with its command.

---

## 6. Cross-edition findings

*Every finding in this section holds in **all three editions**, and each carries the per-edition
locator that proves it. Nothing is placed here on the strength of one edition's evidence.*

### 6.1 A state-changing action is served over `GET`

**Editions: MVC 3, MVC 4 and MVC 5.**

**Finding F-09-24 — `AddToCart` mutates the database over `GET`, in every edition, and no
anti-forgery policy can cover it.** Severity **High**.

The action is declared with **no verb attribute** in all three editions, and each edition's own
comment says what that means:

| Edition | Declaration | Comment above it | It mutates |
| --- | --- | --- | --- |
| MVC 5 | [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:33] | `// GET: /ShoppingCart/AddToCart/5` [:31] | album read [:37-38], cart write [:43], `SaveChanges()` [:45] |
| MVC 4 | [src/MVC4/MvcMusicStore/Controllers/ShoppingCartController.cs:33] | `// GET: /ShoppingCart/AddToCart/5` [:31] | album read [:37-38], cart write [:43], `SaveChanges()` [:45] |
| MVC 3 | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/ShoppingCartController.cs:33] | `// GET: /Store/AddToCart/5` [:31] | album read [:37-38], cart write [:43]; the commit is internal to the cart [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:57] |

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
`Guid.NewGuid()` [:172]. Every cart query filters on that one string —
[src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:37-39], [:64-66], [:89], [:100], [:107], [:120],
[:186].

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
  [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:75-76], [:83].

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
[src/MVC5/MvcMusicStore/Models/Order.cs:12], [:15], [:18], [:64].

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
  [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:51]. MVC 5's ordering — add [:44],
  compute [:48], commit once [:51] — is what makes it safe, and MVC 3's different ordering is what
  makes F-09-22 possible. Removing that one attribute in MVC 5 would make the order total
  client-controlled.
- **The promo code bypasses the model entirely.** It is read straight from the raw form,
  `values["PromoCode"]`, and compared against a compile-time constant
  [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:33-34], [:12] — so it is neither
  validated nor bound, and a value that fails the comparison simply redisplays the view [:36]. There
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
[src/MVC5/MvcMusicStore/MvcMusicStore.csproj:285] and [:19],
[src/MVC4/MvcMusicStore/MvcMusicStore.csproj:350] and [:19], and MVC 3's `<IISUrl>` element is empty
with a plain-HTTP development server port
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:225], [:227-228].

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

**Finding F-09-30 — no edition declares `<machineKey>` or any other key material, so every key that
signs a cookie or an anti-forgery token is generated per process.** Severity **High** for a
multi-instance deployment, **Low** for a single instance.

`git grep -in 'machineKey' -- 'src/**/*.config'` → **0** (§9), across all fifteen application
configuration files. Nor is any key configured in code: the cookie middleware sets only two options
[src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:14-18], and the Forms-based editions configure
nothing beyond the login URL and timeout
[src/MVC4/MvcMusicStore/Web.config:36-37],
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:26-28].

The consequence is the same in each edition, for both of the things that depend on those keys:

- **Authentication tickets and cookies** issued by one process cannot be validated by another, so a
  second instance rejects the first instance's signed-in users, and an application-pool recycle signs
  everyone out.
- **Anti-forgery tokens** — where they are validated at all (§3.7, §4.7) — are likewise
  process-scoped, so a token issued by one instance fails validation at another. On a load-balanced
  deployment without affinity, this turns the eight protected POSTs of MVC 5 into intermittent
  failures, which is the failure mode most likely to be "fixed" by removing the validation.

Together with the in-process session that holds the cart key (§3.8, §6.2), this means none of the
three editions can be scaled out safely as configured — the same fact deliverable 11 owns as a
cloud-readiness blocker and deliverable 06 owns as the target key-store decision. Neither is
re-argued here; what is recorded is that the *current* absence has a security consequence and not
merely an operational one, because the loss lands on ticket and token integrity.

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
`Phone` and `Email` on `Order` — [src/MVC5/MvcMusicStore/Models/Order.cs:23], [:28], [:32], [:36],
[:40], [:45], [:49], [:54], [:61] — with the same shape in
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

Deliverable 06 owns encryption at rest and key management for the target; deliverable 03 owns the
retention decision as a workstream. This document records the present state.

### 6.9 The credential stores themselves are committed to source control

**Editions: MVC 3, MVC 4 and MVC 5.**

**Finding F-09-34 — all three editions' credential databases are tracked in git, with their password
material intact.** Severity **Critical**.

Fourteen database binaries totalling exactly **43,376,640 bytes** are tracked (§9). Three of the
seven `.mdf` files are credential stores — one per edition:

| Edition | Committed credential store | Store type |
| --- | --- | --- |
| MVC 3 | `src/MVC3/MvcMusicStore-Assets/Data/ASPNETDB.MDF` (10,485,760 bytes) with `aspnetdb_log.ldf` | classic ASP.NET Membership |
| MVC 4 | `src/MVC4/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20120831200627.mdf` with its log | SimpleMembership |
| MVC 5 | `src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205.mdf` with its log | ASP.NET Identity 1.0 |

A printable-string inspection of each finds the credential tables and columns present (§9 carries the
probe): MVC 3's contains `aspnet_Membership`, `aspnet_Users`, `aspnet_Roles`, `Password` and
`PasswordSalt`; MVC 4's contains `webpages_Membership`, `webpages_Roles`, `UserProfile`, `Password`
and the literal `Administrator`; MVC 5's contains `AspNetUsers`, `AspNetRoles`, `PasswordHash`,
`SecurityStamp` and the literal `Administrator`. **A string probe is evidence, not proof** — it
cannot distinguish an absent column from one stored in a form the probe does not surface — but taken
with the seeded-account facts each edition's own documentation states [src/MVC5/README.md:75-85], the
conclusion is not in doubt: **the password hashes of the seeded accounts, including the administrator
of F-09-05, are in this repository and in its history.**

Three aggravating facts:

1. **They are tracked despite being excluded.** `.gitignore` lists `App_Data/` at [.gitignore:32] and
   `packages/*` at [.gitignore:15]; an ignore rule cannot untrack a file already added. So this is not
   a deliberate decision anyone made — it is a mistake nobody noticed, which is why deliverable 08
   records it as debt rather than as design.
2. **They are also the only schema evidence for the Identity migration**, which is why they cannot
   simply be deleted without a plan — deliverable 05 depends on extracting the real schema from them.
   The security remediation and the migration requirement are in tension, and that tension belongs to
   deliverable 03's sequencing.
3. **Removal requires history rewriting or explicit acceptance.** Deleting the files from the working
   tree leaves every blob in the object database. Deliverable 08 owns that choice.

### 6.10 `customErrors` never appears as a live element

**Editions: MVC 3, MVC 4 and MVC 5.**

**Finding F-09-35 — error-detail behaviour is left entirely to a framework default that no edition
states, while every edition ships `debug="true"`.** Severity **Medium**.

The element occurs **24** times across the repository and **not once** in a live configuration file:
all 24 sit inside commented example blocks in the six XDT transform files, four per file (§9). Zero
occur in any of the three `Web.config`/`web.config` files that are actually loaded.

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

**Finding F-09-36 — ten external-authentication packages are deployed across two editions to serve
zero enabled providers.** Severity **Medium**.

Every provider registration is commented out. In MVC 5 the four registrations occupy
[src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:22-35] — Microsoft Account, Twitter, Facebook and
Google, each with empty credential arguments. In MVC 4 the same four occupy
[src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs:17-29]. Yet the packages ship: deliverable 02 records
four dormant `Microsoft.Owin.Security.*` provider packages in MVC 5 (F-02-03) and six DotNetOpenAuth
packages in MVC 4 (F-02-08), all deployed to the output directory regardless.

Two distinct risks, and neither is "an attacker can sign in through Facebook":

- **Unused code is deployed code.** Ten assemblies of authentication and protocol-parsing logic are
  present in the deployed application, pinned at 2012–2013 versions that deliverable 02 §8.1 places
  in its aged-dependency class. They are attack surface in the loaded-assembly sense, and they are
  surface nobody is maintaining because nobody believes the feature is on.
- **The application code path for external login is live even though no provider is.** MVC 5's
  external-login actions are routable — [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:200],
  [:209], [:237], [:245], [:265] — and MVC 4's likewise
  [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:228-330]. With no provider registered the
  challenge cannot complete, so this is dead rather than dangerous today; the risk is that enabling
  one provider activates an entire untested account-linking and account-creation path, including the
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
| Every account POST validates its anti-forgery token | MVC 5 (8 of 8), MVC 4 (7 of 7) | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:55], [:88], [:113], [:147], [:199], [:236], [:264], [:301]; [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:35], [:67], [:89], [:119], [:163], [:227], [:271] |
| Sign-out is a token-protected `POST`, posted from a real form | MVC 5, MVC 4 | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:300-301], [src/MVC5/MvcMusicStore/Views/Shared/_LoginPartial.cshtml:4-6]; [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:66-67], [src/MVC4/MvcMusicStore/Views/Shared/_LoginPartial.cshtml:5] |
| The account controller is deny-by-default, with per-action opt-out | MVC 5, MVC 4 | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:15]; [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:16] |
| Sign-in failure does not distinguish an unknown user from a wrong password | all three | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:68]; [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:47]; [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:58] |
| The post-sign-in redirect is guarded against open redirection | all three; MVC 3's is the strictest | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:382-392] via `Url.IsLocalUrl` at [:384]; [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:46-47] adds four further checks |
| The order confirmation page verifies the signed-in user owns the order | all three | [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:71-73]; [src/MVC4/MvcMusicStore/Controllers/CheckoutController.cs:71-73]; MVC 3's equivalent in its own `Complete` action |
| Cart removal is scoped to the caller's own cart, not just the row id | all three | [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:64-66]; [src/MVC4/MvcMusicStore/Models/ShoppingCart.cs:64-66]; [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:63-65]. Partially undone by F-09-26, which is a *read* on the same endpoint |
| The checkout bind list restricts model binding to the nine customer fields | MVC 5, MVC 4 | [src/MVC5/MvcMusicStore/Models/Order.cs:8]; [src/MVC4/MvcMusicStore/Models/Order.cs:8]. This is the control MVC 3 lacks — F-09-22 |
| `Username` and `OrderDate` are additionally set server-side after binding | all three | [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:40-41]; [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/CheckoutController.cs:40-41] |
| Three of the four administration `Find` results are null-checked and return 404 | all three | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:33-35], [:74-76], [:106-108]. The fourth is the Low finding in §6.3 |
| Razor views are never served as content — every path and verb maps to the not-found handler | MVC 5 | [src/MVC5/MvcMusicStore/Views/Web.config:31-32] |
| No connection string stores a database password | all three | [src/MVC5/MvcMusicStore/Web.config:12-13]; [src/MVC4/MvcMusicStore/Web.config:15], [:21]; [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:55-59] |
| No error view in any edition renders an exception, a message or a stack trace | all three | MVC 5's is 9 lines and reads no model property [src/MVC5/MvcMusicStore/Views/Shared/Error.cshtml:7-8]; MVC 4's is 11 lines of generic apology; MVC 3's is 9 lines of the same |
| No raw-output helper is used anywhere — no `Html.Raw`, no `MvcHtmlString.Create` | all three | `git grep -nE 'Html\.Raw\|MvcHtmlString\.Create' -- 'src/**/*.cshtml' 'src/**/*.cs'` → `0` (§9) |
| No raw SQL is executed anywhere, so there is no SQL-injection surface | all three | `git grep -nE 'ExecuteSqlCommand\|SqlQuery\|SqlCommand\|CommandText' -- 'src/**/*.cs'` → `0` (§9). All data access is EF LINQ |
| `HttpContext.Current` is never used, so no ambient-context authorization mistake is possible | all three | `git grep -n 'HttpContext.Current' -- 'src/**/*.cs' 'src/**/*.cshtml'` → `0` (§9) |
| No payment data is collected or stored, in any edition | all three | `Order` has no card, account or payment-token property [src/MVC5/MvcMusicStore/Models/Order.cs:11-66] |
| Provisioning is idempotent per operation | MVC 4 only | [src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:29], [:32], [:35] — and MVC 5's failure to do this is F-09-07 |
| Provider exceptions are mapped to safe messages rather than surfaced | MVC 4 (one of two sites), MVC 3 | [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:105], [:107]; [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:140-143], [:151]. The MVC 4 site that does not is F-09-14 |
| The cart is migrated before the authentication cookie is issued | MVC 3 only | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:43], [:45] — the inverse of MVC 5's F-09-04 |
| The echoed album title is HTML-encoded before entering the response | MVC 3 only | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/ShoppingCartController.cs:68] — the only output-encoding call in the repository, and F-09-23 records its loss |
| The AJAX response is written to the DOM with `.text()`, which does not interpret markup | all three | [src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:28] — which is why F-09-23 is Low rather than a scripting finding |

---

## 8. Finding register

Thirty-seven findings. Severity is defined in §1.6 and assigned on exposure in a hosted,
internet-facing deployment. The **Editions** column is the acceptance criterion for this deliverable:
no row is edition-ambiguous, and no finding is asserted for an edition whose evidence is not cited in
the section named.

| # | Finding | Severity | Editions | Section | Consequence owned by |
| --- | --- | --- | --- | --- | --- |
| F-09-01 | Authorization is five controller-level attributes with no in-action defence in depth | Low | MVC 5, MVC 4 | §3.2, §4.2 | 05 |
| F-09-02 | The entire authentication policy — password, lockout, confirmation, cookie lifetime, `Secure`, `SameSite` — is inherited from framework defaults and stated nowhere | Medium | MVC 5, MVC 4 | §3.3, §4.3 | 05 |
| F-09-03 | There is no account lockout, and the Identity 1.0 schema cannot express one; guessing is unthrottled and, per F-09-32, unrecorded | High | MVC 5 | §3.3 | 05 |
| F-09-04 | Sign-in issues the authentication cookie before the cart write it triggers, across two contexts, with no transaction and no logging | Medium | MVC 5 | §3.4 | 05 |
| F-09-05 | A working administrator username and password are committed in cleartext and republished in the README | **Critical** | MVC 5, MVC 4 | §3.5, §4.5 | 05 |
| F-09-06 | Administrator provisioning runs as `async void`, so a provisioning failure is silent | Medium | MVC 5 | §3.6 | 05 |
| F-09-07 | Provisioning is idempotent overall but not per operation; a partial prior run is never repaired | Medium | MVC 5 | §3.6, §4.6 | 05 |
| F-09-08 | Five of thirteen state-changing POST actions do not validate the anti-forgery token; two of the five emit one that nothing checks | High | MVC 5 | §3.7 | 05 |
| F-09-09 | The runtime identity must hold schema-modifying rights it needs only on first run | Medium | MVC 5 | §3.9 | 06 |
| F-09-10 | The checkout wraps its entire write path in a bare `catch`, discarding the exception and showing no message | Medium | all three | §3.10, §6.8 | 05 |
| F-09-11 | The Forms authentication ticket lifetime is 48 hours, unjustified and unrevocable | Low | MVC 4, MVC 3 | §4.1, §5.3 | 05 |
| F-09-12 | Broader token *emission* in MVC 5 makes a view-level anti-forgery audit actively misleading; emission is not coverage | Informational | MVC 5 vs MVC 4 | §4.7 | — |
| F-09-13 | MVC 4's runtime identity must be able to create databases and tables, and the application does both at runtime | High | MVC 4 | §4.8 | 06 |
| F-09-14 | A raw exception object is placed on a response-bound channel, where any change in model-state rendering turns it into disclosure with no change at the call site | Medium | **MVC 4 only** | §4.9 | 05 |
| F-09-15 | The same line destroys the exception entirely as shipped: not shown, not rethrown, not logged | Medium | **MVC 4 only** | §4.9 | 05 |
| F-09-16 | No credential store is declared, so password, lockout and storage policy are properties of the host and cannot be assessed from the repository | High | **MVC 3 only** | §5.1, §5.3 | 10 (host verification) |
| F-09-17 | Every account is created with the literal recovery question `"question"` and answer `"answer"`; conditional on the host provider's own default posture | High | **MVC 3 only** | §5.1 | 10, 05 |
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
| F-09-28 | A single class-level attribute on the persistence entity is the whole over-posting control at checkout, and the promo code bypasses the model entirely | Medium | MVC 5, MVC 4 | §6.4 | 05, 12 |
| F-09-29 | Nothing requires, redirects to or asserts HTTPS, and no edition emits a single security response header | High | all three | §6.5 | 06, 11 |
| F-09-30 | No `<machineKey>` or key material anywhere, so authentication tickets and anti-forgery tokens are process-scoped | High | all three | §6.6 | 06, 11 |
| F-09-31 | Every edition registers an initializer that drops and recreates the database holding orders and PII whenever the model changes | **Critical** | all three | §6.7 | 05, 06, 08 |
| F-09-32 | No logging construct of any kind exists, so no authentication, authorization, administrative or order event is recorded | High | all three | §6.8 | 06 |
| F-09-33 | Nine personal-data fields are stored in the clear with no retention policy, no deletion path and no access log | High | all three | §6.8 | 06, 03 |
| F-09-34 | All three editions' credential databases are tracked in git with their password material intact | **Critical** | all three | §6.9 | 08, 05 |
| F-09-35 | `customErrors` never appears as a live element, so error-detail behaviour is a framework default nobody stated, while every edition ships `debug="true"` | Medium | all three | §6.10 | 05, 06 |
| F-09-36 | Ten external-authentication packages are deployed across two editions to serve zero enabled providers, with the application code path live | Medium | MVC 5, MVC 4 | §6.11 | 05, 02 |
| F-09-37 | `DeleteConfirmed` is the one administration action that uses its `Find` result without a null check, producing an unhandled exception with no log | Low | all three | §6.3 | 05 |

### 8.1 Distribution

| Severity | Count | Findings |
| --- | --- | --- |
| **Critical** | 3 | F-09-05, F-09-31, F-09-34 |
| **High** | 13 | F-09-03, F-09-08, F-09-13, F-09-16, F-09-17, F-09-18, F-09-21, F-09-22, F-09-24, F-09-29, F-09-30, F-09-32, F-09-33 |
| **Medium** | 16 | F-09-02, F-09-04, F-09-06, F-09-07, F-09-09, F-09-10, F-09-14, F-09-15, F-09-19, F-09-20, F-09-25, F-09-26, F-09-27, F-09-28, F-09-35, F-09-36 |
| **Low** | 4 | F-09-01, F-09-11, F-09-23, F-09-37 |
| Informational | 1 | F-09-12 |

By edition: **13** findings hold in all three editions; **7** are MVC 3-only; **3** are MVC 4-only;
**8** are MVC 5-only; the remainder hold in the MVC 4 / MVC 5 pair. That MVC 3 carries the most
edition-specific findings and MVC 5 the second-most is the concrete form of §2.3's point — the newest
edition is not uniformly the safest, and the migration source is the one whose specific findings the
port must carry forward a fix for.

### 8.2 The three findings that change what other deliverables can assume

Called out separately because each one invalidates an assumption a downstream deliverable would
otherwise make:

1. **F-09-16 and F-09-17 (MVC 3).** No deliverable may state MVC 3's password or lockout policy, and
   no effort estimate may treat MVC 3's identity configuration as known. The host verification
   deliverable 10 owns is a precondition, not a formality — F-09-17 is High precisely because it
   depends on a host setting nobody has read.
2. **F-09-34 with F-09-05.** The committed credential stores are simultaneously a Critical security
   finding and the **only** authoritative schema evidence for the Identity migration deliverable 05
   depends on (§6.9). Remediating one blocks the other unless they are sequenced, which is
   deliverable 03's problem and needs to be on its critical path rather than discovered late.
3. **F-09-24 with F-09-08.** The anti-forgery gap and the mutating `GET` are one workstream, not two,
   and the second is not fixable by policy. Any plan that says "add global anti-forgery validation"
   and stops has not addressed `AddToCart` — deliverable 05 must carry the verb change as a separate,
   approval-owned interface delta.

---

## 9. Reproducibility appendix

Every count and every absence claimed above is reproduced here. Commands are run from the repository
root. Those prefixed `git` were run in Git Bash on the Windows host; the PowerShell block covers the
three measurements git alone cannot make.

```bash
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
git grep -in 'machineKey'   -- 'src/**/*.config' | wc -l    # 0
git grep -in 'sessionState' -- 'src/**/*.config' | wc -l    # 0

# --- Observability: absent everywhere -------------------------------------
git grep -inE 'ILogger|log4net|NLog|Serilog|TraceSource|System\.Diagnostics\.Trace|healthMonitoring|EventLog' \
    -- 'src/**/*.cs' 'src/**/*.cshtml' 'src/**/*.config' | grep -v '/packages/' | wc -l                # 0

# --- customErrors: 24 occurrences, all commented, none live ---------------
git grep -c 'customErrors' -- 'src/**/*.config'    # 4 in each of the 6 Web.Debug/Web.Release files = 24
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
# stored procedure named aspnet_Roles_CreateRole. That is not source code — and it is one more
# corroboration of F-09-34, since it is only visible because the credential store is committed.
git grep -I -inE 'CreateAdminUser|CreateRole|DefaultAdminPassword' -- 'src/MVC3/' | wc -l         # 0

# --- Dependency-scanning configuration: none ------------------------------
git ls-files | grep -E '^\.github/|dependabot|renovate' | wc -l    # 0
```

```powershell
# --- The 14 committed database binaries and their exact total -------------
$f = git ls-files | Where-Object { $_ -match '\.(mdf|ldf)$' -or $_ -match '\.(MDF|LDF)$' }
$f.Count                                                                       # 14
($f | ForEach-Object { (Get-Item $_).Length } | Measure-Object -Sum).Sum        # 43376640

# --- Credential-store string probe (EVIDENCE, NOT PROOF) -----------------
# A printable-string probe cannot distinguish an absent column from one stored
# in a form the probe does not surface. Read UTF-16 as well as ASCII: SQL Server
# stores object names as nvarchar, so an ASCII-only probe finds nothing.
function Probe($path, $patterns) {
  $bytes = [System.IO.File]::ReadAllBytes($path)
  $ascii = [System.Text.Encoding]::ASCII.GetString($bytes)
  $utf16 = [System.Text.Encoding]::Unicode.GetString($bytes)
  foreach ($p in $patterns) { "{0,-22} ascii={1,-5} utf16={2}" -f $p, $ascii.Contains($p), $utf16.Contains($p) }
}
Probe 'src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205.mdf' `
      @('AspNetUsers','AspNetRoles','PasswordHash','SecurityStamp','Administrator',
        'LockoutEnabled','AccessFailedCount','TwoFactorEnabled','EmailConfirmed')
#   AspNetUsers/AspNetRoles/PasswordHash/SecurityStamp/Administrator -> utf16=True
#   LockoutEnabled/AccessFailedCount/TwoFactorEnabled/EmailConfirmed -> not found  (F-09-03)
Probe 'src/MVC4/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20120831200627.mdf' `
      @('webpages_Membership','webpages_Roles','UserProfile','Password','Administrator')   # all utf16=True
Probe 'src/MVC3/MvcMusicStore-Assets/Data/ASPNETDB.MDF' `
      @('aspnet_Membership','aspnet_Users','aspnet_Roles','Password','PasswordSalt')        # all True

# --- F-09-14 / F-09-15: what Html.ValidationSummary actually renders -----
# Compiled and run against MVC 4's OWN committed assembly, so the result is this
# edition's behaviour and not a guess about some version of MVC.
$mvc = 'src/MVC4/MvcMusicStore/packages/Microsoft.AspNet.Mvc.4.0.20710.0/lib/net40/System.Web.Mvc.dll'
# harness: ViewDataDictionary + ModelState.AddModelError("", new Exception("MARKER"))
#          then a real HtmlHelper over a real ViewContext, rendering
#          ValidationExtensions.ValidationSummary(html) and (html, true).
# Observed:
#   ModelError.ErrorMessage           = ""   (length 0); ModelError.Exception retained
#   ValidationSummary()               = <div class="validation-summary-errors"><ul><li style="display:none"></li></ul></div>
#   ValidationSummary(true)           = identical
#   MARKER present in either output   = False
#   CONTROL, string overload          = <li>MARKER</li>  -> present = True
# The harness lived outside the checkout and is not committed; the repository was
# not modified to run it, and `packages/` was read, never written.
```

### 9.1 The constraint this work was held to

No repository file was modified, added or deleted in the course of this assessment other than the
deliverables under `docs/modernization/`. Every command above is read-only. The ad-hoc harness of
§9 was compiled and run outside the checkout, reading MVC 4's committed package payload and writing
nothing into it. The verification is the same one AAP §0.11.5 makes the final acceptance criterion:

```bash
git status --porcelain    # only new files under docs/modernization/
```

### 9.2 Secondary cross-reference

Technical Specification §6.4 describes the same three parallel authentication stacks, the same
inheritance of password and session policy from framework defaults, the same per-edition anti-forgery
coverage, the same five authorization enforcement points, and the same verified absence of audit
logging, application cryptography and transport protection. It corroborates this assessment and is
cited nowhere in place of the repository evidence that establishes each finding.

Where this document and any other input disagree, the repository governs — which is why §4.9 reports
what the shipped assembly does rather than what a plausible reading of one line suggests, and why
§5.1 declines to state a password policy that the checkout does not contain.

---

*Deliverable 09 of 13. Consumes 01 (architecture) and 02 (dependencies); feeds 12 (migration
blockers) alongside 10 and 11. Every finding cites a repository path and locator and names the
editions it holds in; every count and absence carries the command that reproduces it. **No repository
file was changed** — the defects recorded here are documented for approval, and remediation begins
only once the assessment and modernization plan are approved.*
