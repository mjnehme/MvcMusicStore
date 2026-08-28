# 05 — ASP.NET Core Migration Approach

Deliverable 05 of 13. Consumes deliverables [12](12-migration-blockers.md),
[04](04-dotnet8-migration-strategy.md), [09](09-security-assessment.md),
[11](11-cloud-readiness-assessment.md) and [08](08-technical-debt-register.md); feeds
[03](03-modernization-roadmap.md) and [07](07-effort-risks-sequencing.md). Index and requirement map:
[README](README.md).

---

## 1. Purpose, ownership and authoring contract

### 1.1 What this document is

This is the **specification for porting MvcMusicStore's MVC 5 edition from ASP.NET MVC 5 on
System.Web to ASP.NET Core**. It is written to be executed: an engineer holding this document, its
sibling deliverables and repository access should be able to carry out the port without making a
design decision this document declined to make.

Deliverable [12](12-migration-blockers.md) enumerated twenty-two blockers and, for each, named the
successor or recorded that there is none. **This document's job is the next step and a different
one: every one of those twenty-two gets a resolution — a decision, with the target construct named,
the affected sites listed, and the test that proves it.** Section [13](#13-resolution-register--all-22-blockers-of-deliverable-12)
walks all twenty-two and is the audit trail for that claim.

The scope is the **migration source only**. Edition triage is [03](03-modernization-roadmap.md)'s to
sequence and it selected MVC 5; MVC 4 and MVC 3 are retained read-only as historical references and
as the behavioural baseline, and appear here only where their code is evidence about a decision this
document makes — which happens once, in section [10](#10-administrator-provisioning-becomes-an-operator-command).

### 1.2 What this document is not

It is not an inventory. Where a fact about the current system is already established and owned
elsewhere, this document **cites it and moves to the resolution** rather than re-deriving it. It is
not a schedule: no duration, no sequence position and no effort figure appears anywhere below, because
those belong to [07](07-effort-risks-sequencing.md) and [03](03-modernization-roadmap.md) and a second
statement of them would be a second decision.

And it is not authorization. See section [1.4](#14-the-no-modification-constraint-and-the-boundary-that-makes-this-document-possible).

### 1.3 Authoring contract, and the absence of user rules

**`review_rules` returns exactly "No user rules provided."** Verified directly during the production
of this document. There is therefore no project rule to name, summarize or cite, and no rule forcing a
file into scope. That absence is **not** licence to lower the bar; enterprise-standard practice applies
instead, and this document holds itself to four explicit contracts:

1. **Citation contract.** Every claim about the **existing** system carries an inline
   `[<path>:<locator>]` citation at the point of the claim, with a repository-relative path that
   resolves in the checkout. A prescription about the target needs no repository citation — but the
   fact that motivates it almost always does, and where it does, it carries one.
   **Shorthand convention:** a bare `[:N]` or `[:N-M]` locator refers to the **most recently named
   file in the same sentence, bullet, table row or table caption**. Where that antecedent would be
   even slightly ambiguous, the full path is repeated instead.
2. **Repository-wide claims carry their reproducing command.** A count or an absence has no single
   line to point at, so its evidence is the command that produces it. Every such command in this
   document was run against this checkout; [Appendix A](#appendix-a--reproducibility) collects them.
3. **Exact versions only, and version pins are not this document's to state.** Where a version
   matters, [04 §7.2](04-dotnet8-migration-strategy.md) and [04 §9.2](04-dotnet8-migration-strategy.md)
   own the pins. This document names the *capability* and cites 04 for the *version*.
4. **One fact, one owner.** Section [1.6](#16-what-this-document-does-not-own) lists what belongs to
   someone else. Where this document needs such a fact, it links to the owner rather than restating
   the value.

### 1.4 The no-modification constraint, and the boundary that makes this document possible

The user directed **"Do not make code changes initially"**, and the project's environment setup
instructions independently restate the same gate. Nothing in this repository was created, modified or
deleted in producing this document except the file you are reading; the acceptance check is that
`git status --porcelain` shows only new files under `docs/modernization/`.

**This document lives closer to that line than any other in the set, so the line is worth stating
precisely: the prohibition is on *mutating*, not on *planning*.**

Every target artifact named below — `Program.cs`, `appsettings.json`, `ErrorViewModel.cs`,
`Binding/CheckoutInputModel.cs`, `Services/ShoppingCartService.cs`, `Data/MusicStoreEntities.cs`,
`Data/ApplicationDbContext.cs`, `Data/Migrations/Catalog/**`, `Data/Migrations/Identity/**`,
`Data/SeedData.cs`, the three `*ViewComponent.cs` classes and their `Default.cshtml` views,
`Views/_ViewImports.cshtml`, `wwwroot/**`, `libman.json`, `src/MvcMusicStore.Tests/**` and
`tools/provision-admin/**` — is **content written into this document, not a file created on disk.**
Likewise, every row below whose fate is "Deleted" describes a target state; nothing was deleted.

**This document therefore fails in both directions.** It fails if it reads as authorization to begin
changing code. It fails equally **if it declines to specify a change because it mistook the
prohibition on mutating for a prohibition on planning** — and that second failure is the more likely
one, because it looks like caution. What follows is written as a specification an engineer executes
after approval.

### 1.5 What this document owns

| Decision | Section |
| --- | --- |
| **The cutover approach and its accepted losses** — sole owner under the one-fact-one-owner rule | [11](#11-the-cutover-decision) |
| The composition root: what replaces `Global.asax.cs`, `Startup.cs` and the five `App_Start` files | [2](#2-the-composition-root) |
| The configuration-reading transition, and which half of the XDT transform goes where | [3](#3-configuration-and-the-one-active-transform) |
| Dependency injection, context lifetimes, and the cross-store consistency sequence | [4](#4-dependency-injection-and-object-lifetime) |
| Whether the two `DbContext` classes remain separate | [4.5](#45-two-contexts-two-migration-sets-two-history-tables) |
| Per-site lazy-loading resolutions | [4.6](#46-lazy-loading--a-resolution-per-site-not-a-policy) |
| Schema extraction as a gate, the migration design, and the seeding guard | [5](#5-schema-lifecycle-and-the-two-data-migrations) |
| The Identity schema-and-data migration | [5.5](#55-the-identity-transition--a-schema-decision-plus-a-data-migration) |
| Authentication, cookie, password and lockout policy, each labelled preserved or hardening | [6](#6-authentication-policy-is-decided-not-inherited) |
| The anti-forgery policy, the `AddToCart` verb change and the AJAX token contract | [7](#7-anti-forgery--three-separate-problems) |
| Static-asset delivery, the five legacy-type views, the view components, the Bootstrap markup port | [8](#8-views-static-assets-and-the-wire-contracts) |
| Whether the disabled external-login surface is mapped or removed | [8.3](#83-the-five-views-that-name-legacy-types) and [13](#13-resolution-register--all-22-blockers-of-deliverable-12) |
| The JSON naming resolution, scoped or global | [8.7](#87-the-json-contract--annotate-one-model-not-the-policy) |
| The checkout input model | [8.8](#88-the-checkout-input-model--ten-properties-not-nine) |
| Administrator provisioning as an operator command | [10](#10-administrator-provisioning-becomes-an-operator-command) |
| The test-suite architecture and coverage, and the separate manual visual review | [12](#12-the-test-suite-architecture-and-coverage) |

### 1.6 What this document does not own

| Fact | Owner | How this document treats it |
| --- | --- | --- |
| Target framework, SDK band, project format | [04 §2](04-dotnet8-migration-strategy.md), [04 §3](04-dotnet8-migration-strategy.md), [04 §5](04-dotnet8-migration-strategy.md) | Cites. Names no framework version and no SDK band. |
| Every package pin, including client-side libraries, `libman.json` and the test-framework pins | [04 §7.2](04-dotnet8-migration-strategy.md), [04 §9](04-dotnet8-migration-strategy.md) | Cites. Names capabilities, never versions. |
| Hosting target, deployment model, cutover runbook, browser matrix | [06](06-azure-hosting-recommendations.md) | Cites. |
| **Observability**: the telemetry collection mechanism | [06](06-azure-hosting-recommendations.md) | Records `ILogger` usage **in application code** only. Does not state how logs are collected. |
| Data-protection key-ring location and rotation | [06](06-azure-hosting-recommendations.md), context in [11 §3.2](11-cloud-readiness-assessment.md) | Cites. |
| The deployment-time DDL mechanism and the principal that holds DDL rights | [06](06-azure-hosting-recommendations.md) | Cites. States the **ordering**, which is a migration-design fact. |
| Session cache table provisioning | [06](06-azure-hosting-recommendations.md) | Cites. |
| Per-edition build outcomes, including MVC 5's verified build and the clean-checkout restore precondition that survives it | [10 §3.1](10-build-and-deployment-requirements.md), [10 §5.4](10-build-and-deployment-requirements.md) | Cites. |
| Effort model, risk register, the visual-review task | [07](07-effort-risks-sequencing.md) | Cites. States no figure. |
| Workstream sequencing and gate placement | [03](03-modernization-roadmap.md) | Cites. |
| Statefulness assessment and the path-casing finding of record | [11 §3.1](11-cloud-readiness-assessment.md), [11 §3.7](11-cloud-readiness-assessment.md) | Cites. |
| The blockers themselves, and the line-counting methods | [12](12-migration-blockers.md), [08 §2.1](08-technical-debt-register.md) | Cites. Resolves rather than re-enumerates. |

---

## 2. The composition root

### 2.1 Disposition of every current startup and configuration file — twelve files, four fates

Composition in MVC 5 is split across twelve files and two entry points. In the target it is one file
plus a handful of Razor and MSBuild conventions. The table below is the **complete** disposition, and
its column headings matter: *Physical fate* is what happens to the file, *Responsibility* is what
happens to what the file did. Those are different questions, and four distinct answers appear in the
second column.

**"No counterpart" is not the same as "no successor."** A responsibility with a successor moves; a
responsibility with no counterpart **ends**, and the target simply does not have the concern. Both
appear below and each row says which it is.

| Current file | Physical fate | Responsibility |
| --- | --- | --- |
| `Global.asax.cs` [src/MVC5/MvcMusicStore/Global.asax.cs:11-21] | Deleted | **Moves, per registration** — the five registrations do not share one fate; see [2.2](#22-the-five-application_start-registrations-do-not-share-one-fate) |
| `Startup.cs` [src/MVC5/MvcMusicStore/Startup.cs:4-14] | Deleted | **Moves** — attribute-declared host discovery becomes explicit composition in `Program.cs`; see [2.3](#23-owin-startup-the-abstraction-goes-the-responsibilities-do-not) |
| `App_Start/RouteConfig.cs` [src/MVC5/MvcMusicStore/App_Start/RouteConfig.cs:12-21] | Deleted | **Moves** — one conventional route registration in `Program.cs`; the `.axd` ignore route ends |
| `App_Start/FilterConfig.cs` [src/MVC5/MvcMusicStore/App_Start/FilterConfig.cs:8-11] | Deleted | **Replaced** — exception-handling middleware plus the global anti-forgery filter of section [7](#7-anti-forgery--three-separate-problems) |
| `App_Start/Startup.App.cs` [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:14-45] | Deleted | **Splits** — the initializer registration [:16] is replaced by migrations; administrator provisioning [:18-45] **leaves the web application entirely** for `tools/provision-admin` (section [10](#10-administrator-provisioning-becomes-an-operator-command)) |
| `App_Start/Startup.Auth.cs` [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:11-36] | Deleted | **Moves and becomes explicit** — cookie authentication is a framework feature; the policy it silently inherited today is stated in section [6](#6-authentication-policy-is-decided-not-inherited); the external sign-in cookie [:20] and the four commented-out providers [:23-35] **end**, per the decision in section [8.3](#83-the-five-views-that-name-legacy-types) |
| `Global.asax` — the markup file [src/MVC5/MvcMusicStore/Global.asax:1] | Deleted | **Ends.** The target has no application-declaration file of any kind and nothing reads this one |
| `App_Start/BundleConfig.cs` [src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs:9-29] | Deleted | **Ends.** No bundler in the target — section [8.1](#81-bundling-and-static-assets--no-bundler) |
| `Views/Web.config` [src/MVC5/MvcMusicStore/Views/Web.config] | Deleted | **Splits.** Namespace registration [:14-21] moves to `Views/_ViewImports.cshtml`; the `BlockViewHandler` mapping [:31-32] **ends** — section [8.6](#86-view-serving-and-_viewimports) |
| `Properties/AssemblyInfo.cs` [src/MVC5/MvcMusicStore/Properties/AssemblyInfo.cs:8-15] | Deleted | **Absorbed** into MSBuild properties — [04 §5.3](04-dotnet8-migration-strategy.md) owns it |
| `Web.Debug.config` | Deleted | **Ends.** It carries no active transform at all — verified, section [3.2](#32-the-one-active-transform-and-the-two-halves-it-splits-into) |
| `Web.Release.config` [src/MVC5/MvcMusicStore/Web.Release.config:18] | Deleted | **Re-expressed.** Its one active transform becomes a publish-time Release setting — section [3.2](#32-the-one-active-transform-and-the-two-halves-it-splits-into) |

Two rows in that table are the ones a reader is most likely to compress into the others, so they are
called out. `Startup.App.cs` **splits across a project boundary** — half of it becomes migrations and
half of it leaves the web application to become a separate console tool — which is a stronger statement
than "moves into `Program.cs`". And `Views/Web.config` splits too, with one half having a direct
successor and the other having no counterpart; a plan that deletes the file without noticing the
namespace half loses six `@using` directives that 29 views depend on.

### 2.2 The five `Application_Start` registrations do not share one fate

`public class MvcApplication : System.Web.HttpApplication`
[src/MVC5/MvcMusicStore/Global.asax.cs:11] runs five registrations in `Application_Start`
[src/MVC5/MvcMusicStore/Global.asax.cs:13-21]. Treating the method as one unit is the single most
common way to get this port wrong, because **three different outcomes are present in five lines**.

| Registration | Line in `Global.asax.cs` | Resolution | Kind |
| --- | --- | --- | --- |
| `AreaRegistration.RegisterAllAreas()` | [:15] | **Deleted.** There is no `Areas` folder anywhere in the repository — `git ls-files \| grep -c 'Areas/'` returns `0` — so the call has always been dead scaffolding ([08 §9.1](08-technical-debt-register.md) owns it as debt). Nothing replaces it. | Ends |
| `FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters)` | [:16] | **Replaced.** Its one filter, `new HandleErrorAttribute()` [src/MVC5/MvcMusicStore/App_Start/FilterConfig.cs:10], becomes **exception-handling middleware** — a pipeline registration, not a filter — and the global filter slot it vacates is taken by the **anti-forgery filter** of section [7](#7-anti-forgery--three-separate-problems). Design in section [8.3](#83-the-five-views-that-name-legacy-types). | Replaced |
| `RouteConfig.RegisterRoutes(RouteTable.Routes)` | [:17] | **Moves.** The `.axd` ignore route [src/MVC5/MvcMusicStore/App_Start/RouteConfig.cs:14] is **dropped** — the target has no `.axd` handler, so there is nothing to exclude. The single conventional route [:16-20] becomes one conventional route registration with the pattern `{controller=Home}/{action=Index}/{id?}`, which preserves the whole of the application's URL surface: one route, no areas, no attribute routing, no constraints. | Moves |
| `BundleConfig.RegisterBundles(BundleTable.Bundles)` | [:18] | **Deleted outright**, under the no-bundler decision of section [8.1](#81-bundling-and-static-assets--no-bundler). The five bundle virtual paths cease to exist, which is an approved delta (section [11.5](#115-the-full-set-of-approved-deltas)). | Ends |
| `System.Data.Entity.Database.SetInitializer(new SampleData())` | [:20] | **Replaced** by explicit migrations plus the guarded seeding command of section [5.4](#54-seeding--the-guard-fails-closed-on-three-checks). Note this call is made a **second** time at [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:16] — `SetInitializer<TContext>` *sets* rather than *adds*, so only one initialization ever runs ([08 §5.1](08-technical-debt-register.md) owns the redundancy). Both call sites disappear into one migration path, which is the point. | Replaced |

### 2.3 OWIN startup: the abstraction goes, the responsibilities do not

MVC 5 has a **second** entry point, declared by assembly attribute:
`[assembly: OwinStartupAttribute(typeof(MvcMusicStore.Startup))]`
[src/MVC5/MvcMusicStore/Startup.cs:4]. Its `Configuration(IAppBuilder app)` [:9] calls `ConfigureAuth`
[:11] and then `ConfigureApp` [:13].

**`IAppBuilder` has no successor type.** Nothing in the target has that shape, so the three method
signatures that take it — [src/MVC5/MvcMusicStore/Startup.cs:9],
[src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:14] and
[src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:11] — cannot be preserved even in principle, and the
`Owin` and `Microsoft.Owin.*` package family goes with it ([04 §8.2](04-dotnet8-migration-strategy.md)
assigns each package its outcome). Attribute-based host discovery goes too: composition in the target is
explicit in the program entry point, so **there is no attribute and no convention to find it by.**

What survives is every *responsibility*: cookie authentication, the database bootstrap decision, and
administrator provisioning. Each is re-expressed rather than translated.

### 2.4 What `Program.cs` contains, in order

Six files collapse into one composition root. Order matters twice over — service registration must
precede use, and middleware order is behaviour, not style — so this is specified as an ordered
sequence rather than a list of ingredients.

**Stage 1 — service registration (the container).** In this order:

1. **Configuration binding.** Options types bound from `IConfiguration` using the options pattern
   (section [3.1](#31-configuration-webconfig-becomes-appsettingsjson-read-through-iconfiguration)).
2. **The two `DbContext` registrations**, scoped, each with its own connection and its own migrations
   assembly and history table (section [4.5](#45-two-contexts-two-migration-sets-two-history-tables)).
3. **Identity**, over the Identity context, with **every policy value set explicitly** — password,
   lockout, confirmation, user options (section [6](#6-authentication-policy-is-decided-not-inherited)).
4. **Authentication and the cookie handler**, with lifetime, sliding expiration, `SameSite`, `Secure`
   and the login path all set explicitly. The login path preserves `/Account/Login`
   [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:17].
5. **Data protection**, with its key ring persisted rather than left per-instance — the store's
   location is [06](06-azure-hosting-recommendations.md)'s, the *requirement* is
   [11 §3.2](11-cloud-readiness-assessment.md)'s.
6. **The distributed cache and session**, session keyed as today by the constant at
   [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:19] (section [6.3](#63-session-and-cart-identity)).
7. **Application services** — `ShoppingCartService` scoped
   (section [4.8](#48-the-service-layer-and-the-two-patterns-deliberately-not-adopted)).
8. **MVC**, with **two** global filters: the anti-forgery filter of section
   [7](#7-anti-forgery--three-separate-problems), and nothing corresponding to `HandleErrorAttribute`,
   which becomes middleware instead.
9. **Anti-forgery options**, including the header name of section [7.4](#74-problem-three--cart-removal-posts-without-a-form).
10. **Health checks**, including a database probe — net-new capability; the repository has no health
    endpoint of any kind ([11 §3.8](11-cloud-readiness-assessment.md)).

**Stage 2 — the middleware pipeline.** In this order:

1. **Exception handling.** The developer exception page **only** outside production, and the
   exception-handler middleware forwarding to the error route otherwise. This is where
   `HandleErrorAttribute` lands, and section [8.3](#83-the-five-views-that-name-legacy-types) specifies
   the route, the status-code behaviour, what is logged and what may be disclosed.
2. **HSTS and HTTPS redirection.** Net-new: the application is served over plain HTTP today with no
   security response header of any kind ([11 §3.6](11-cloud-readiness-assessment.md)).
3. **Static files**, serving `wwwroot` (section [8.1](#81-bundling-and-static-assets--no-bundler)).
4. **Routing.**
5. **Session** — before authentication is not required, but it must precede any component that reads
   the cart key, and the cart key is read on every page from the layout.
6. **Authentication**, then **authorization**.
7. **The conventional route registration** — `{controller=Home}/{action=Index}/{id?}`.

**Migrations are not applied here.** No migration call, no `EnsureCreated`, no seeding and no
provisioning appears in `Program.cs`. Section [5.3](#53-who-applies-ddl-and-in-what-order) states why
and what replaces it.

---

## 3. Configuration and the one active transform

### 3.1 Configuration: `Web.config` becomes `appsettings.json` read through `IConfiguration`

Two `connectionStrings` entries [src/MVC5/MvcMusicStore/Web.config:12-13] and six `appSettings` keys
[:15-22] are the whole of the application's configuration surface. In the target they become
`appsettings.json` and `appsettings.Development.json`, read through `IConfiguration`, with typed
options classes bound at startup.

**The options pattern is required here rather than merely conventional, and the reason is a specific
line of code.** `ConfigurationManager.AppSettings` is read **directly from startup code**
[src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:23-24], via `using System.Configuration;` [:7]. That
is not a configuration *file* problem that a file format solves — it is a startup component reaching
into a static ambient configuration store, which the target has no equivalent of. The two values it
reads are the administrator credential, and section
[10](#10-administrator-provisioning-becomes-an-operator-command) removes them from configuration
altogether, so the correct target for [:23-24] is **nothing at all**: no `appsettings` key, no
options class, no environment variable in the web application.

Per-key disposition. Every bare `[:N]` locator below refers to
`src/MVC5/MvcMusicStore/Web.config`:

| Current key | Target |
| --- | --- |
| `connectionStrings/DefaultConnection` [src/MVC5/MvcMusicStore/Web.config:12] | One connection string for one database, supplied by platform configuration — the mechanism and the identity model are [06](06-azure-hosting-recommendations.md)'s; the blocker framing is [11 §3.4](11-cloud-readiness-assessment.md)'s |
| `connectionStrings/MusicStoreEntities` [src/MVC5/MvcMusicStore/Web.config:13] | The **same** connection string; see section [4.5](#45-two-contexts-two-migration-sets-two-history-tables) for why one database and what it trades |
| `DefaultAdminUsername`, `DefaultAdminPassword` [:16-17] | **Removed from source entirely.** Section [10](#10-administrator-provisioning-becomes-an-operator-command). [09 §3.5](09-security-assessment.md) owns the finding |
| `webpages:Version`, `webpages:Enabled` [:18-19] | **End.** Web Pages is not part of the target; the same `webpages:Enabled=false` appears redundantly at [src/MVC5/MvcMusicStore/Views/Web.config:26] and ends with it |
| `ClientValidationEnabled`, `UnobtrusiveJavaScriptEnabled` [src/MVC5/MvcMusicStore/Web.config:20-21] | **End as configuration keys.** Both behaviours are the target's default and are not switched from `appsettings` |
| `system.web/authentication mode="None"` [src/MVC5/MvcMusicStore/Web.config:32] | **Ends.** The value is an artifact of authentication having been moved to OWIN; the target has no `system.web` |
| `system.webServer/modules remove FormsAuthenticationModule` [src/MVC5/MvcMusicStore/Web.config:38] | **Ends.** An IIS-integrated-pipeline instruction with no counterpart |
| `compilation targetFramework="4.8"` [src/MVC5/MvcMusicStore/Web.config:33] versus `httpRuntime targetFramework="4.5"` [:34] | **Ends — the concept has no target.** The target has no quirks mode, so there is nothing to carry the value into. The consequence is for the *baseline*, not the port: see below |

**The framework-version mismatch is a baseline-integrity problem, and it must be handled before the
baseline is captured rather than diagnosed afterwards.** `httpRuntime targetFramework="4.5"` [:34]
holds 4.5-era compatibility behaviour in force even though the project targets 4.8
([12 F-12-18](12-migration-blockers.md) owns the finding). So the behaviour a reviewer records from
the running MVC 5 application **is 4.5 quirks-mode behaviour**, and the ported application has no
quirks mode at all. The resolution is procedural and belongs to the baseline exercise of section
[12](#12-the-test-suite-architecture-and-coverage): the baseline capture must record that the source
runtime was in 4.5 quirks mode, and any behavioural difference the suite surfaces in request
validation, encoding or a related default must be checked against that setting **before** it is
classified — because the error is available in both directions. A pre-existing configuration artifact
can be misfiled as a port regression, and a real regression can be dismissed as "just the quirks
mode".

### 3.2 The one active transform, and the two halves it splits into

Across the six transform files in the repository there are **fifteen** `xdt:Transform` occurrences and
only **three** of them are outside an XML comment — one per `Web.Release.config`, and all three are
the same line at the same line number:

```text
src/MVC5/MvcMusicStore/Web.Release.config:18:  <compilation xdt:Transform="RemoveAttributes(debug)" />
src/MVC4/MvcMusicStore/Web.Release.config:18:  <compilation xdt:Transform="RemoveAttributes(debug)" />
src/MVC3/.../MvcMusicStore/Web.Release.config:18:  <compilation xdt:Transform="RemoveAttributes(debug)" />
```

The three `Web.Debug.config` files carry **zero** active transforms. Every other occurrence in all six
files is commented-out template text. The verifying command is in
[Appendix A](#appendix-a--reproducibility).

**This one transform splits into two target concerns, and conflating them is the error this section
exists to prevent.**

**Half one — build and publish.** Removing the `debug` attribute from `<compilation>` is what stops
the release deployment compiling with debug semantics. Its target is the **Release build
configuration's own publish behaviour**: `dotnet publish -c Release`, under which optimization and
debug-symbol behaviour are properties of the configuration. It does **not** map to an `appsettings`
key and it does **not** map to an environment variable. There is nothing to write into a configuration
file for it, which is why a reader looking for its successor in `appsettings.json` will not find one
and may wrongly conclude the behaviour was lost.

**Half two — runtime error behaviour.** What `debug="true"` *also* influenced in practice — how much
detail an error response discloses — is in the target a **runtime** decision made in the middleware
pipeline: the developer exception page is registered only outside production, and the
exception-handler middleware handles everything else (section
[2.4](#24-what-programcs-contains-in-order), designed in section
[8.3](#83-the-five-views-that-name-legacy-types)). That is governed by the environment the application
runs in, not by the build configuration it was compiled with. **The two are independently settable in
the target and must be set independently** — a Release build can run in a Development environment and
a Debug build in Production, and neither combination should surprise anyone reading this plan.

**One net-new obligation follows from the same six files.** The commented-out examples include a
connection-string substitution that was **never active**, so the repository has no working mechanism
for supplying a secret at deploy time. Providing one is therefore net-new work rather than a migration
of anything, and its mechanism is [06](06-azure-hosting-recommendations.md)'s.

---

## 4. Dependency injection and object lifetime

### 4.1 The ten manual instantiation sites constructor injection replaces

Ten sites in the migration source construct a `DbContext` or an Identity manager by hand. They are
enumerated individually because they are not all the same shape and two of them are not in a
controller at all.

| # | Site | What is constructed | Target |
| --- | --- | --- | --- |
| 1 | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:19] | `new UserManager<ApplicationUser>(new UserStore<ApplicationUser>(new ApplicationDbContext()))` — a chained constructor building the entire Identity graph, three objects deep | The chained constructor is deleted; `UserManager` and `SignInManager` are injected. The second, parameter-taking constructor already at [:23-26] is the shape that survives |
| 2 | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:32] | `new MusicStoreEntities()` — one ad hoc catalog context **inside a method body**, in `MigrateShoppingCart` | Injected catalog context, or the injected `ShoppingCartService` that supersedes this method's body (section [4.8](#48-the-service-layer-and-the-two-patterns-deliberately-not-adopted)) |
| 3 | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:15] | `MusicStoreEntities` field initializer | Constructor-injected scoped context |
| 4 | [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:12] | `MusicStoreEntities` field initializer | Constructor-injected scoped context |
| 5 | [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:10] | `MusicStoreEntities` field initializer | Constructor-injected scoped context |
| 6 | [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:11] | `MusicStoreEntities` field initializer | Constructor-injected scoped context |
| 7 | [src/MVC5/MvcMusicStore/Controllers/HomeController.cs:11] | `MusicStoreEntities` field initializer | Constructor-injected scoped context |
| 8 | [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:27] | `new ApplicationDbContext()` at startup | **Leaves the web application.** Resolved from the console tool's host — section [10](#10-administrator-provisioning-becomes-an-operator-command) |
| 9 | [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:28] | `new UserManager<ApplicationUser>(new UserStore<ApplicationUser>(context))` at startup | As above |
| 10 | [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:29] | `new RoleManager<IdentityRole>(new RoleStore<IdentityRole>(context))` at startup | As above |

**`AccountController` is the exception in this list, and describing it as "the sixth controller with a
field-initialized context" is wrong.** It has **no** field-initialized catalog context. It holds an
`ApplicationDbContext` only *indirectly*, through the `UserManager` its chained constructor builds
[:19], and it creates exactly **one** `MusicStoreEntities` on demand, inside `MigrateShoppingCart`
[:32]. So dependency injection does two different things to this file: it replaces that single ad hoc
catalog context, and it replaces the hand-built Identity graph with injected managers. The Identity
context remains a **separate registration** either way — it is not folded into the catalog context.

The file also already has the constructor injection needs: `public AccountController(UserManager<ApplicationUser> userManager)`
[:23-26] exists today and is unused by the framework, because MVC 5 resolves the parameterless one.
The port keeps that shape and deletes the parameterless overload.

### 4.2 What scoped registration actually changes — and what it does not

**Scoped registration gives one instance per context type per request, not one instance overall.**
That sentence is the whole of this section, and it is here because the opposite reading produces a plan
that quietly assumes atomicity it will not have.

After the port there are still **two `DbContext` instances per request** and therefore **two change
trackers** — one for `MusicStoreEntities`, one for `ApplicationDbContext` — and **no automatic unit of
work spanning them**. A `SaveChanges` on one does not enlist the other.

What scoped registration removes is narrower and worth having:

- the hand-rolled construction at all ten sites in section [4.1](#41-the-ten-manual-instantiation-sites-constructor-injection-replaces);
- the ad hoc mid-method instance at [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:32],
  which today means a *second* catalog context exists inside a request that already has none of its
  own — the account controller has no field context, so this instance is created, used and abandoned;
- the consumer-side disposal that becomes wrong (section [4.7](#47-the-dispose-overrides-must-be-removed));
- and the inconsistency where one controller's context is disposed and four others' are not
  ([08 §5.8](08-technical-debt-register.md) owns that as debt).

What it does **not** remove is the two-store boundary. That boundary is a design decision, taken
deliberately in section [4.5](#45-two-contexts-two-migration-sets-two-history-tables), and the
consistency model that makes it safe is section [4.3](#43-the-cross-store-consistency-model).

### 4.3 The cross-store consistency model

Sign-in writes to the Identity store; cart reassignment writes to the catalog store. And **a database
transaction cannot make an authentication cookie atomic with either of them**, because the cookie is
issued in the HTTP response, outside any transaction and outside the database entirely. So the
ordering has to be designed rather than inherited.

**The current order is the reverse of the safe one, which is the evidence that this needs deciding.**
`SignInAsync` [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:346-354] signs out the external
cookie [:348], creates the identity [:349], **registers the sign-in grant** [:350], and only **then**
migrates the cart [:353]. `MigrateShoppingCart` [:30-40] creates its own context [:32], reassigns
ownership through `MigrateCart` [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:184-193], calls
`SaveChanges` [:37] and rewrites the session key [:39]. The action that calls it
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:63] has no `try`/`catch`, so a failure in the
cart write propagates out of a method that has already granted the sign-in.

**The target sequence, in order:**

1. **Authenticate and complete all Identity-side writes first**, including any password rehash
   (section [5.5](#55-the-identity-transition--a-schema-decision-plus-a-data-migration)), and **commit
   them**.
2. **Then migrate the cart** in the catalog store, in **its own transaction**.
3. **Then issue the authentication cookie** — after both commits have succeeded.
4. **Cart migration is idempotent and keyed on the anonymous cart id**, so a retry cannot double-assign
   rows. Retry on transient failure.
5. **If cart migration fails after retries, complete the sign-in anyway** and surface a non-blocking
   notice to the user. A signed-in user with an unmigrated cart is recoverable — they can add the items
   again, and the orphaned rows are reportable. A failed sign-in is not recoverable by the user at all.

**The invariant this achieves, stated so a reviewer can check it: the cart is never reassigned to a
user who is not signed in.** Step 3 comes after step 2, so a cart carrying a username always
corresponds to a completed authentication. The converse — a signed-in user whose cart did not
migrate — is permitted by step 5, deliberately, because it is the recoverable direction.

**Both failure paths need tests**: cart migration failing after retries must still produce a
successful sign-in with the notice, and an Identity-side failure must produce **no** cart reassignment
at all.

**Merging the two contexts into one was considered and is not taken.** It would collapse this ordering
problem into a single `SaveChanges` and would be the simpler design on that axis alone. It is rejected
because `ApplicationDbContext : IdentityDbContext<ApplicationUser>`
[src/MVC5/MvcMusicStore/Models/IdentityModels.cs:10-16] is a framework-shaped context whose model the
Identity packages own, and merging the six catalog `DbSet` properties
[src/MVC5/MvcMusicStore/Models/MusicStoreEntities.cs:7-12] into it entangles the catalog model with
Identity's schema versioning — every future Identity package upgrade would then arrive as a migration
against the catalog. The ordering above is five lines of explicit code; the merge is a permanent
coupling. **The ordering is cheaper than the coupling, so the boundary stays.** Note also that the
merge would not have removed the cookie problem, which is not a database problem at all.

### 4.4 What "the cart" is coupled to — and what it is not

Stated here because sections [4.3](#43-the-cross-store-consistency-model) and
[4.5](#45-two-contexts-two-migration-sets-two-history-tables) both depend on it.

`Order.Username` is a plain string [src/MVC5/MvcMusicStore/Models/Order.cs:17-18], compared against
`User.Identity.Name` [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:71-73]. The cart's
`CartId` is likewise a plain string, holding either the signed-in user's name or a generated GUID
[src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:161-180]. **Neither references the Identity primary
key, and there is no foreign key between the catalog store and the credential store in either
direction.** The two stores are coupled **only by convention** — a string that happens to match a login
name.

That fact carries three consequences used later: it is why one database is an operational choice rather
than a relational necessity ([4.5](#45-two-contexts-two-migration-sets-two-history-tables)); it is why
**usernames** rather than user ids are the invariant of the Identity data migration
([5.5](#55-the-identity-transition--a-schema-decision-plus-a-data-migration)); and it is why signed-in
carts survive cutover while anonymous ones do not ([11.4](#114-two-accepted-costs)).

### 4.5 Two contexts, two migration sets, two history tables

**Neither context's current configuration convention survives.**

`MusicStoreEntities : DbContext` declares six `DbSet` properties, **no constructor and no
`OnModelCreating`** [src/MVC5/MvcMusicStore/Models/MusicStoreEntities.cs:5-13]. EF 6 resolves its
connection string by matching the **class name** against a `connectionStrings` entry, and there is one
named `MusicStoreEntities` [src/MVC5/MvcMusicStore/Web.config:13]. The coupling between the class and
its database is a name, expressed nowhere in the code — which is why this is the dangerous half: the
class compiles in the target, instantiates in the target, and simply does not know where its database
is. There is no line for a reviewer to notice, because the thing that stops meaning something is the
**absence** of a constructor.

`ApplicationDbContext : IdentityDbContext<ApplicationUser>` passes the name explicitly,
`: base("DefaultConnection")` [src/MVC5/MvcMusicStore/Models/IdentityModels.cs:12-13], matching the
entry at [src/MVC5/MvcMusicStore/Web.config:12]. This half announces itself — the base overload does
not exist in the target, so it fails to compile. ([12 F-12-19](12-migration-blockers.md) owns the
as-is finding and its two opposite failure modes.)

**Resolution.** Both contexts get an **explicit `DbContextOptions<T>` constructor** and an explicit
container registration, scoped, with the connection supplied from configuration. `MusicStoreEntities`
additionally gains an `OnModelCreating` override, because EF 6 inferred its entire mapping by
convention and several of those conventions differ in the target — key discovery on the `RecordId`,
`AlbumId`, `GenreId`, `ArtistId`, `OrderId` and `OrderDetailId` properties, `decimal` precision and
scale on `Album.Price`, `Order.Total` and `OrderDetail.UnitPrice`, and the delete behaviour on each
relationship. **Every one of those is settled against the extracted schema of section
[5.1](#51-domain-data-migration-starts-with-schema-extraction), not against a convention.**

**The two contexts remain separate.** [12 §7.3](12-migration-blockers.md) records this as 05's
decision; it is taken, for the reason in section [4.3](#43-the-cross-store-consistency-model).

**Both target one database.** [06](06-azure-hosting-recommendations.md) owns the hosting and
provisioning; the *reason* belongs here, and it must be stated accurately because the obvious
justification is false. **It is not a foreign key between them — there is none** (section
[4.4](#44-what-the-cart-is-coupled-to--and-what-it-is-not)). Consolidation therefore removes an
**operational** dependency, not a relational one, and the trade is:

| Gained | Given up |
| --- | --- |
| One connection string to configure, rotate and secure | A **shared blast radius** — a bad migration or an exhausted resource affects credentials and catalog together |
| One migration target and one deployment-time DDL step | **Coarser permission granularity** — both contexts and the session cache share one principal's grants, so the catalog code path has whatever access the credential tables grant |
| One backup, restore and point-in-time-recovery story, with catalog and credentials consistent to the same instant | **A single scaling unit** — the catalog's read volume and the credential store's write volume cannot be scaled apart |
| One instance to provision, monitor and pay for | |

For an application of this size with one deployable unit, operational simplicity wins. The trade is
recorded so a reader can reverse it knowingly.

**Each context owns a separate migrations folder and a distinct migrations history table** —
`Data/Migrations/Catalog/**` and `Data/Migrations/Identity/**`, with two differently-named history
tables. This is not tidiness: with a shared history table, one context's migrations appear as
**pending** to the other, and a tool run against either context reports a state that is wrong for it.
Two folders and two history tables make each context's migration state independently and correctly
readable.

### 4.6 Lazy loading — a resolution per site, not a policy

EF 6 populates navigation properties on demand through proxies. The target does not: an unloaded
reference navigation is `null`. The code that depends on this **compiles identically under both**, so
nothing announces the change.

**The decision is explicit eager loading or projection at each query site — not a lazy-loading proxy
package.** Two reasons: the affected set is small and fully enumerated below, so explicit loading is
*verifiable* in a way a global proxy behaviour is not; and a proxy package would re-create the exact
property that makes this blocker silent — a query whose loading behaviour is invisible at the call
site.

"Use eager loading" is a policy, not a plan. [12 F-12-15](12-migration-blockers.md) enumerated
**nine** sites in **three categories that need opposite treatment**. Each gets its own resolution.

**Why exactly these sites and no others.** EF 6 lazy loading requires the navigation to be `virtual`.
`Album.Genre` [src/MVC5/MvcMusicStore/Models/Album.cs:30], `Album.Artist` [:31],
`Album.OrderDetails` [:32] and `Cart.Album` [src/MVC5/MvcMusicStore/Models/Cart.cs:17] are `virtual`;
`Genre.Albums` [src/MVC5/MvcMusicStore/Models/Genre.cs:10] is **not**, which is why the genre-browse
page had to eager-load and is therefore unaffected.

**Category (a) — client-side dereference after materialization. Six sites; every one is a definite
break. Resolution: eager-load or project at the query.**

| # | Query | Dereference | Resolution |
| --- | --- | --- | --- |
| a1 | `storeDB.Albums.Find(id)` [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:38] | `@Model.Genre.Name` [src/MVC5/MvcMusicStore/Views/Store/Details.cshtml:16], `@Model.Artist.Name` [:20] | Replace `Find` with a filtered single-result query carrying **`Include(a => a.Genre)` and `Include(a => a.Artist)`**. `Find` cannot take an `Include`, so the method call changes, not just its arguments |
| a2 | `db.Albums.Find(id)` [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:32] | `model.Artist.Name` [src/MVC5/MvcMusicStore/Views/StoreManager/Details.cshtml:18], `model.Genre.Name` [:26] | Same change as a1. Note the sibling **list** action at [:22] already uses the typed `Include(a => a.Genre).Include(a => a.Artist)` and ports unchanged — which is why the admin list page is absent from this table and the admin detail page is in it |
| a3 | `storeDB.Carts.Single(item => item.RecordId == id).Album.Title` [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:61-62] | `.Album.Title` on the same expression [:62] | **Project instead of loading**: select the title directly in the query, so no navigation is dereferenced client-side at all. This also fixes the unscoped read that [09 §6.2](09-security-assessment.md) records, because the projection carries the cart-id predicate |
| a4 | `GetCartItems()` — `ToList()` with no `Include` [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:98-101] | `.Select(a => a.Album.Title)` [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:91-92] | The cart-summary path gets its **own projection query** returning count and titles, rather than materializing `Cart` entities and walking navigations. This is the highest-reach site: it renders from the shared layout [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:26], so a failure is **every page**, not one page |
| a5 | `GetCartItems()` via the cart view model [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:17-24] | `item.Album.Title` [src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:64], `@item.Album.Price` [:68] | The cart-page query carries **`Include(c => c.Album)`**. This is the one site where the view genuinely needs the entity graph, so `Include` rather than projection |
| a6 | `GetCartItems()` inside order creation [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:129] | `orderTotal += (item.Count * item.Album.Price)` [:145] | **The financial one.** Resolution: compute the running total from `album.Price` — the album the same loop **already loads explicitly** at [:134] and already uses for `UnitPrice` at [:140] — instead of from the lazy navigation at [:145]. One loop, two ways of reaching the same price, and only one of them survives; the surviving one is already in the file. Acceptance is an order-total assertion in section [12.4](#124-required-coverage) |

**Category (b) — navigation read inside a server-translated query. One site. Resolution: verify the
translation; do *not* add an `Include`.** `GetTotal` composes
`select (int?)cartItems.Count * cartItems.Album.Price` over the `DbSet` and calls `.Sum()` on the
resulting queryable [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:119-122]. The navigation is read
inside an expression the provider translates to SQL, so **no lazy load ever occurs** and the target
translates it too. Adding an `Include` here would load entities the query does not need and would be a
regression in the name of a fix. The required work is a **test that the query translates and returns
the same total**, not a code change. Listing this alongside category (a) would send the port to the
wrong fix, which is why the category exists.

**Category (c) — nested collection aggregates. Two sites. Resolution: verify translation, and rewrite
to a translatable form if it fails.** The genre menu orders by a nested `Sum` over each genre's
albums' order-detail quantities [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:46-52], and the
home page orders by `a.OrderDetails.Count()`
[src/MVC5/MvcMusicStore/Controllers/HomeController.cs:29-32]. **The failure here is louder than a
null**: EF 6 silently client-evaluates parts of a query it cannot translate, whereas the target
**throws**. So the risk is an exception at a query that used to work — and the genre menu, like a4,
renders on every page from the layout [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:25].
Resolution in order of preference: confirm the aggregate translates; if it does not, rewrite it as an
explicit grouped projection over `OrderDetails`; and in either case add the caching that
[08 §5.2](08-technical-debt-register.md) records as separate performance debt, because both queries
run on every page. **Both sites must have their generated SQL inspected once**, not merely their output
asserted, since a query that throws and a query that is slow present very differently.

**One favourable detail that reduces the work.** The **string-based** `Include("Albums")`
[src/MVC5/MvcMusicStore/Controllers/StoreController.cs:30] **remains valid** in the target, even
though the typed lambda form is the convention. It may be modernized for consistency; it does not have
to be.

### 4.7 The `Dispose` overrides must be removed

Two overrides in the migration source dispose objects the code currently constructs and therefore
currently owns:

| Override | Object disposed | Constructed at |
| --- | --- | --- |
| [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:125-128] | `MusicStoreEntities` | field initializer [:15] |
| [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:324-331] | `UserManager<ApplicationUser>` | chained constructor [:19] |

([12 F-12-20](12-migration-blockers.md) records four across all three editions; the two above are the
migration source's.)

Today each is correct — the controller creates the object, so the controller disposes it, and the
account controller is even careful about it, null-guarding and nulling the field [:326-330].

**Resolution: delete both overrides.** There is no replacement construct; container-owned lifetimes
need no disposal from the consumer. **The reason this is a required step rather than a tidy-up is that
leaving them compiles.** `Dispose(bool)` still exists on the target's controller base class, so there
is no signature error and no warning — and then a scoped object is disposed before the scope ends,
producing an `ObjectDisposedException` at a call site that has nothing wrong with it. Two properties
make it hard to catch: the failure is **order-dependent**, so a single-request test may not reproduce
it, and it surfaces away from its cause.

The account controller carries the sharper version, because its `UserManager` is one of three objects
its chained constructor builds [:19]. Once the container supplies all three, an override disposing the
middle one reaches an object other consumers may still hold. That override is deleted along with the
constructor, in the wholesale rewrite of section [9.2](#92-the-four-groups).

### 4.8 The service layer, and the two patterns deliberately not adopted

**`ShoppingCart` stops being a model and becomes a service.** Its business logic is real:
`CreateOrder` computes the order total, writes `OrderDetail` rows, sets `order.Total` and empties the
cart [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:125-158], and `MigrateCart` reassigns cart
ownership on sign-in [:184-193]. That is the one place in the application where business logic sits in
a model, so it is the one place worth extracting — to `Services/ShoppingCartService.cs`, registered
scoped, taking the catalog context and the HTTP context accessor by injection.

Three changes travel with the move:

1. **`HttpContextBase` becomes the framework's context type.** The class takes it as an explicit
   parameter [:21] and reads session through it [:161-180]; it never touches a static ambient context.
   That is the favourable finding of [12 P-12-01](12-migration-blockers.md) — `HttpContext.Current`
   appears nowhere in the repository — and it is why this move is a signature change rather than an
   architectural excavation.
2. **The dead `Controller` overload is dropped.**
   `public static ShoppingCart GetCart(MusicStoreEntities db, Controller controller)` [:29-32] takes a
   `System.Web.Mvc.Controller`, a type with no target equivalent — and it is **provably unreferenced**.
   All six call sites use the `HttpContextBase` overload [:21]:
   [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:35],
   [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:17], [:41], [:58], [:89] and
   [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:47]. **Deleting it has zero call-site
   impact**, which is materially cheaper than porting it ([12 P-12-02](12-migration-blockers.md)).
3. **The static factory pair collapses.** With the service injected and the cart id resolved from the
   accessor, `GetCart` stops being a static factory taking a context and becomes ordinary instance
   state.

**Two patterns are deliberately not adopted, and the reasons are specific to this application rather
than general.**

**No repository abstraction over EF Core.** `DbContext` is already a unit of work and `DbSet<T>` is
already a queryable collection; the domain has six entities
[src/MVC5/MvcMusicStore/Models/MusicStoreEntities.cs:7-12] and there is exactly one data source and no
outbound service dependency ([11 §4.3](11-cloud-readiness-assessment.md)). A repository layer would add
indirection with no test seam `DbContext` does not already provide — and the suite of section
[12](#12-the-test-suite-architecture-and-coverage) is HTTP-level black-box, so it does not need one.
The one place a genuine seam is wanted is the cart's business logic, and that is what
`ShoppingCartService` is.

**No uniform service layer across the six controllers.** Five of the six are thin: they query, build a
view model and return a view. `HomeController` is 34 physical lines
([08 §3.1](08-technical-debt-register.md)). Wrapping each in a service would produce five pass-through
classes and one real one. The real one is extracted; the pass-throughs are not written.

---

## 5. Schema lifecycle and the two data migrations

### 5.1 Domain data migration starts with schema extraction

**An EF Core initial migration creates empty tables. It moves no rows.** That sentence needs saying
because "generate the initial migration" reads like the data step and is not it.

Worse, **the generated migration cannot be trusted to match the database the data is actually in.** A
migration generated from the ported model may differ from the real EF 6 schema in column type,
precision and length, nullability, identity and key definition, delete rule, default value and index —
and none of those differences produces a build error or a startup error. They produce wrong data, or a
failed insert, at the first write. **And there is nothing in the repository to compare against:**
MVC 5 ships no schema script at all, and MVC 4's two copies are byte-identical and not runnable as
written ([12 F-12-22](12-migration-blockers.md) owns both facts; [04 §13.1](04-dotnet8-migration-strategy.md)
carries the instruction that neither may be used as a baseline).

**Therefore the domain-data workstream begins with extraction, and the diff is a gate.** In order:

1. **Authoritative schema extraction from `src/MVC5/MvcMusicStore/App_Data/MvcMusicStore.mdf`.** A
   `sys.columns`, `sys.indexes`, `sys.key_constraints` and `sys.foreign_keys` interrogation of the
   attached database, producing a written schema record for all six catalog tables. This can only be
   done on a supported Windows and LocalDB runtime, because that is the only place the file attaches
   ([10](10-build-and-deployment-requirements.md) owns the toolchain). **It must be done against a
   copy outside the checkout** — attaching the tracked `.mdf` and `.ldf` causes the engine to write to
   them, which would dirty the working tree.
2. **Map every extracted object to the EF Core model**, and express the result in
   `MusicStoreEntities.OnModelCreating` (section [4.5](#45-two-contexts-two-migration-sets-two-history-tables)).
3. **Generate the initial catalog migration and diff its DDL against the extracted schema. The diff
   must pass — meaning zero unexplained differences — before any data is loaded.** Each difference is
   either corrected in the model or recorded as a deliberate schema change with a reason. This is the
   gate; a data load against an unverified schema is the failure this whole section exists to prevent.
4. **Back up the source, then extract the data.** The source `.mdf` is preserved unmodified as the
   rollback position.
5. **Apply the baseline migration to the target database**, then **load in dependency order**:
   **genres and artists** (no dependencies), then **albums** (referencing both), then **carts**
   (referencing albums), then **orders**, then **order details** (referencing orders and albums).
   Identity-column behaviour must be handled explicitly at load time so that existing key values are
   preserved — `Cart.RecordId`, `Order.OrderId` and `OrderDetail.OrderDetailId` are referenced by
   other rows and by nothing else, but `Order.OrderId` is also the **confirmation number** the
   application shows the customer [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:53-54],
   so renumbering orders would change a value users have already been given.
6. **Reconcile.** Row count **per table**, before and after. And **financial totals per order** —
   `Order.Total` [src/MVC5/MvcMusicStore/Models/Order.cs:63-64] compared against the sum of its
   `OrderDetail.UnitPrice * Quantity` rows, computed on the source and on the target and required to
   match to the cent. Reconciliation failure blocks cutover.
7. **Protect the data in transit and at rest.** The order tables carry name, postal address, phone
   number and email address [src/MVC5/MvcMusicStore/Models/Order.cs:20-61], so the extract is PII: it
   travels only over an encrypted connection, is held only in an encrypted location, is access-logged,
   and is destroyed on a stated schedule after reconciliation passes. [09 §6.8](09-security-assessment.md)
   owns the standing PII finding.
8. **Define how rows written between extraction and cutover are handled.** Under the single-cutover
   decision of section [11](#11-the-cutover-decision) the answer is a **drain**: the legacy
   application is drained before the final extract, so the window is closed rather than
   reconciled. Any trickle-load design is therefore explicitly **not** built; if the cutover approach
   were ever revisited, this step would have to be re-answered.
9. **State the rollback position and the closing acceptance criteria.** Rollback is: the source
   database is unmodified, the legacy application is redeployable, and the target database is
   discarded. Acceptance is: the schema diff passed, every per-table row count matches, every order
   total matches, and the coverage of section [12.4](#124-required-coverage) passes against the loaded
   data.

### 5.2 The seed is not the data migration

`SampleData` seeds 15 genres, roughly 303 artists and roughly 462 albums
[src/MVC5/MvcMusicStore/Models/SampleData.cs:9-20]. It is tempting to treat "apply migrations, then
run the seed" as the data step, and that would be wrong in a way that loses every order and cart: the
seed writes **catalog reference data only** and knows nothing of `Cart`, `Order` or `OrderDetail`. The
seed is a development convenience (section
[5.4](#54-seeding--the-guard-fails-closed-on-three-checks)); section
[5.1](#51-domain-data-migration-starts-with-schema-extraction) is the data migration. They are
different workstreams with different targets and must not be substituted for one another.

### 5.3 Who applies DDL, and in what order

**Migrations are not applied by the web application at startup, and not under the runtime identity.**
No migration call appears in `Program.cs` (section [2.4](#24-what-programcs-contains-in-order)). Two
reasons, both structural: a startup migration means every replica races to apply DDL on deployment,
and a runtime identity holding DDL rights means an application-level compromise can drop tables.

**[06](06-azure-hosting-recommendations.md) owns the deployment-time mechanism and the principal that
holds DDL rights.** This document does not restate them.

**What this document does own is the ordering, because it is a property of the migration design.** Four
schema owners must be created in a fixed order, and each step's success is the next step's
precondition:

| Order | Schema owner | Created by | Why here |
| --- | --- | --- | --- |
| 1 | **The session cache table** | The SQL cache tool of [04 §6.3](04-dotnet8-migration-strategy.md), run once | Session backs the cart identity on every request, and no application code path can create it. It is first because it is outside both migration sets and nothing depends on it |
| 2 | **The catalog context's migrations** | `Data/Migrations/Catalog/**` | The domain data load targets these tables |
| 3 | **The Identity context's migrations** | `Data/Migrations/Identity/**` | Separate history table, so step 2 and step 3 never report each other as pending |
| 4 | **The data-protection key table** | The key-ring store of [06](06-azure-hosting-recommendations.md) | Must exist before the first request, or the application generates an ephemeral key ring and cookies stop surviving a restart |
| 5 | **The domain data load** (section [5.1](#51-domain-data-migration-starts-with-schema-extraction)) | The migration workstream | After 2 |
| 6 | **The Identity data migration** (section [5.5](#55-the-identity-transition--a-schema-decision-plus-a-data-migration)) | The migration workstream | After 3 |

Seeding does not appear in that list. It is never part of a production release path — section
[5.4](#54-seeding--the-guard-fails-closed-on-three-checks).

### 5.4 Seeding — the guard fails closed on three checks

`SampleData : DropCreateDatabaseIfModelChanges<MusicStoreEntities>`
[src/MVC5/MvcMusicStore/Models/SampleData.cs:9] is two problems in one declaration. The base type
**drops and recreates the database whenever the model changes** — against a database holding orders
and PII ([08 §6.1](08-technical-debt-register.md), [09 §6.7](09-security-assessment.md)). And the
`Seed` override [:11-20] is roughly 820 non-blank lines of hardcoded catalog rows
([08 §4.2](08-technical-debt-register.md) owns the count and its method).

**Resolution, in two independent parts.**

**The destructive initializer is not reproduced.** Nothing in the target drops or recreates a database.
Schema change is migrations only, applied per section [5.3](#53-who-applies-ddl-and-in-what-order).

**The seed becomes `Data/SeedData.cs`, invoked by an explicit opt-in command — not automatically at
startup.** `HasData` is impractical at this row count, because it would place ~780 rows into the model
snapshot and make every subsequent migration diff unreadable. So it is a routine — and the routine
needs a guard.

**`IsDevelopment()` alone is not a guard**, and this is the point of the section. The environment name
and the connection string are configured **independently**, so a misconfigured environment variable —
exactly the most common deployment mistake — produces a process that believes it is in Development
while pointing at the production database. The seed would then run against real data.

**Three checks, all of which must pass, evaluated before any write:**

1. **The environment must not be Production.** Necessary, not sufficient.
2. **An explicit enable flag must be set** — a configuration value whose only purpose is to authorize
   seeding, absent by default and never set in any deployed configuration. This defeats the
   misconfigured-environment case, because an environment variable that is wrong by accident does not
   also set a flag that exists for no other reason.
3. **The target database name must match an allow-listed non-production pattern**, read from the
   connection the context is actually about to use. This is the check that binds the decision to the
   *database* rather than to the process's belief about itself, and it is the one that catches the
   failure the other two miss.

**It fails closed**: any check failing aborts with a non-zero exit and **writes nothing**. Deployment
ordering places seeding after migrations, in non-production only, and **never in the production release
path**.

**The guard itself is a test.** Section [12.4](#124-required-coverage) requires that a seeding attempt
against a production-shaped configuration fails and leaves the database byte-unchanged — asserted by
row counts before and after, not merely by an exit code.

### 5.5 The Identity transition — a schema decision plus a data migration

**The schema decision: create the target's Identity tables fresh and populate them, rather than
altering the 1.0 tables in place.** Two reasons. It leaves the source tables intact as a rollback
position, so a failed migration is reversible by pointing at the old store rather than by restoring a
backup. And it lets reconciliation compare **two live datasets** rather than a dataset against a log.

**Extraction is a gate here too, and it is the same gate.** A `sys.columns` query against the attached
Identity database must complete and be reconciled against the target Identity model **before any
Identity data is migrated**. [12 F-12-21](12-migration-blockers.md) owns the evidence and — importantly
— owns its **qualification**: a printable-string probe of the committed credential store finds the
early Identity column names and none of the six later ones, but **string-probing a binary is evidence,
not proof**, and a negative result cannot distinguish an absent column from one the probe does not
surface. This document does not re-derive the probe and does not treat its negative result as proof.
It requires the authoritative query.

Note the sequencing collision [09 §8.2](09-security-assessment.md) records and
[03](03-modernization-roadmap.md) must place: **the committed credential stores are simultaneously the
only authoritative schema evidence and a Critical security finding**, so removing them and extracting
from them are the same artifact's two futures and have to be ordered.

**The data migration, specified.**

**User names, not user ids, are the invariant — and the two must not be conflated.**
`Order.Username` stores the **login name** [src/MVC5/MvcMusicStore/Models/Order.cs:17-18] and is
compared against `User.Identity.Name`
[src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:71-73]. The cart's `CartId` likewise holds a
login name [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:167]. **Neither references the Identity
primary key.** So:

- **Preserving usernames exactly is a hard requirement.** A changed username orphans that user's order
  history and their cart, silently — the query returns no rows rather than failing.
- **Preserving the original id values is a separate and optional choice.** It **is** taken here, for
  **convenience rather than referential necessity**: keeping the ids makes role assignments and
  external-login rows trivially re-linkable during the load, and makes a row-by-row comparison between
  source and target possible. Nothing in the application would break if ids changed. The distinction
  matters because a plan that treats id preservation as a correctness requirement will spend effort
  defending it, and a plan that treats username preservation as optional will lose order history.

**Normalization is a new column and a new collision risk.** `NormalizedUserName` and
`NormalizedEmail` do not exist in the 1.0 schema. Normalization is typically an upper-casing, so **two
accounts differing only in case normalize to the same value** and the second insert violates a unique
index. **The migration detects collisions before writing and stops** — it enumerates the source
usernames and emails, computes their normalized forms, and aborts on any duplicate, reporting the
colliding accounts for a human decision. It does **not** de-duplicate, rename or drop an account
automatically; silently losing an account is the outcome this check exists to prevent.

**Fields with no source value get a defined origin.** Each is stated rather than left to whatever the
insert happens to produce:

| Target column | Origin |
| --- | --- |
| `Id` | Carried from the source, per the optional-but-taken choice above |
| `UserName` | Carried exactly. Invariant |
| `NormalizedUserName` | Computed from `UserName` by the target's normalizer, after the collision check |
| `Email` | Carried where present. The 1.0 store may hold none |
| `NormalizedEmail` | Computed where `Email` is present; **null** where it is not — nullable, not empty string, so "no email" stays distinguishable from "empty email" |
| `EmailConfirmed` | **`false`**, documented. Not `true`: no confirmation ever occurred, and asserting otherwise would be inventing a security fact |
| `PasswordHash` | Carried verbatim. See below |
| `SecurityStamp` | **Generated fresh for every account.** A new stamp invalidates nothing that survives cutover, since every session ends anyway (section [11.4](#114-two-accepted-costs)) |
| `ConcurrencyStamp` | **Generated fresh for every account** |
| `PhoneNumber`, `PhoneNumberConfirmed` | **null** and **`false`** — no source value exists |
| `TwoFactorEnabled` | **`false`** — the source has no two-factor concept |
| `LockoutEnabled` | **`true`** — this is a **deliberate hardening**, labelled as such in section [6.1](#61-the-policy-table-every-row-labelled), because there is no lockout today at all ([09 §3.3](09-security-assessment.md), finding F-09-03) |
| `LockoutEnd` | **null** — no account starts locked out |
| `AccessFailedCount` | **`0`** |

**Password hashes are verified, not assumed.** The target's default password hasher can validate the
older hash format and rehash on a successful sign-in — but that is a property to **confirm against the
hashes this repository actually ships**, not to assert. The verification is concrete: after the load,
a sign-in as a pre-existing account must succeed, and the stored hash must be observed to have been
rewritten in the target's current format. **The acceptance test is a successful sign-in by a
pre-existing account** (section [12.4](#124-required-coverage)). If verification fails, the fallback is
a forced password reset for all migrated accounts — which is a materially different user-facing
outcome and therefore an approval decision, not an implementation detail, so it is surfaced rather than
absorbed.

**Reconciliation and rollback.**

- Account count, role count and role-assignment count compared **before and after**.
- **The administrator's role membership asserted specifically**, by name, not merely counted — it is
  the one assignment whose loss makes the entire administration surface unreachable, which is exactly
  the failure MVC 3 ships as a shipped state ([09 §5.2](09-security-assessment.md)).
- A per-account comparison of username and hash between source and target.
- **Source tables retained until reconciliation passes.** They are the rollback position, and they are
  dropped only as a separate, later step under the security remediation that
  [09 §8.2](09-security-assessment.md) sequences.

---

## 6. Authentication policy is decided, not inherited

Cookie authentication is configured today with **exactly two values** — an authentication type
[src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:16] and a login path [:17], inside the options object
at [:14-18] — plus the external sign-in cookie [:20]. Nothing else is set anywhere. **The policy is
therefore not absent: it is in force on every request and it is whatever the framework's defaults
happen to be.** [09 §3.3](09-security-assessment.md) owns that finding (F-09-02) and assigns this
document the duty of setting each value explicitly.

**The principle: hardening is welcome; hardening that arrives as an unannounced framework default is a
change nobody approved.** The target's defaults are mostly better than the source's. That is not a
licence to leave them implicit — an unstated value cannot be reviewed, cannot be approved, and
silently changes again on the next framework upgrade. **Every row below is set explicitly in
`Program.cs`, and every row is labelled.**

### 6.1 The policy table, every row labelled

| Policy | Today | Target | Label |
| --- | --- | --- | --- |
| Password minimum length | `UserManager` default; no `PasswordValidator` assigned anywhere in the edition | Set explicitly, at or above the target framework's default | **Deliberate hardening** if raised above the source's effective value; **preserved** if equal. Either way, stated |
| Password complexity (digit, lowercase, uppercase, non-alphanumeric, unique characters) | `UserManager` defaults | Each of the five requirements set explicitly | **Deliberate hardening** |
| Account lockout enabled | **No lockout exists** — sign-in is a single `FindAsync` call with no attempt counter, no delay and no lockout [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:60], and the failure path only adds a message and redisplays [:68] | Enabled | **Deliberate hardening** — and the highest-value one in the table. [09 §3.3](09-security-assessment.md) F-09-03 rates it High |
| Lockout threshold and duration | n/a | Both set explicitly | **Deliberate hardening** |
| Account confirmation required for sign-in | Identity 1.0 default; **no confirmation flow is implemented** and no email is sent | **Not required.** Requiring it would lock out every migrated account, since `EmailConfirmed` is `false` for all of them (section [5.5](#55-the-identity-transition--a-schema-decision-plus-a-data-migration)) and many hold no email at all | **Preserved** — deliberately, with the reason recorded, because this is the one row where the hardening is the wrong call |
| Unique email required | Identity 1.0 default | **Not required.** The source may hold accounts with no email; enforcing uniqueness at migration time would fail the load | **Preserved** |
| Cookie lifetime | `CookieAuthenticationOptions` default, unset [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:14-18] | Set explicitly | **Preserved** where the value matches the source's effective default; stated regardless |
| Sliding expiration | Middleware default, unset | Set explicitly | **Preserved**, stated |
| Persistent-cookie duration ("remember me") | Middleware default; the flag is honoured at [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:63] | Set explicitly; the "remember me" capability is **preserved** | **Preserved**, with the duration now stated |
| Cookie `Secure` attribute | Katana default `SameAsRequest`, which over plain HTTP means **not secure** ([09 §3.3](09-security-assessment.md), [11 §3.6](11-cloud-readiness-assessment.md)) | **Always secure** | **Deliberate hardening** |
| Cookie `HttpOnly` | Middleware default (on) | Set explicitly, on | **Preserved**, stated |
| Cookie `SameSite` | **Not an option in the source middleware at all** | Set explicitly to `Lax`, which preserves top-level navigation to the site while blocking cross-site sub-requests | **Deliberate hardening**. `Strict` is not chosen: it would break inbound links from external pages into authenticated views |
| Login path | `/Account/Login` [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:17] | `/Account/Login` | **Preserved** |
| Access-denied path | Not configured — the source redirects to login for both cases | Set explicitly | **Deliberate hardening**, minor: an authenticated non-administrator gets a denial rather than a login form |
| External-login surface | Scaffolded and **entirely disabled** — all four provider registrations commented out [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:23-35] — plus the external sign-in cookie [:20] | **Removed** — see section [8.3](#83-the-five-views-that-name-legacy-types) and the resolution of `ChallengeResult` in section [13](#13-resolution-register--all-22-blockers-of-deliverable-12) | **Deliberate change.** No user-visible capability is lost, because none was reachable ([09 §6.11](09-security-assessment.md) records it as deployed attack surface) |
| Data-protection key store | **None declared** — no `machineKey` in any edition, so key material is per-instance and ephemeral ([11 §3.2](11-cloud-readiness-assessment.md)) | Persisted key ring. **Location, protection at rest, rotation and slot isolation are [06](06-azure-hosting-recommendations.md)'s** | **Deliberate hardening** |

Two rows deserve a sentence beyond the table. **The confirmation and unique-email rows are the two
places where this document declines a hardening**, and both declines are consequences of the Identity
data migration rather than of taste — which is why they are recorded here with their reason instead of
being quietly omitted. And **the `SameSite` row has no "today" value at all** to be preserved or
hardened relative to, because the source middleware has no such option; it is stated as hardening
because the resulting behaviour is stricter than the absence.

### 6.2 The sign-out surface

Sign-out today is a POST with anti-forgery validation
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:300-306], posted by a form in the shared login
partial [src/MVC5/MvcMusicStore/Views/Shared/_LoginPartial.cshtml:4-14]. **That is preserved
exactly** — a POST, token-validated, from a form. It is called out because MVC 3 ships a `GET`
sign-out ([09 §5.4](09-security-assessment.md)) and a reader comparing editions could conclude the
verb needs changing here. It does not.

The one change is mechanical: the link at [:12] invokes
`javascript:document.getElementById('logoutForm').submit()`, which an inline-script Content Security
Policy would block. Since the port adds security response headers where the source has none
([11 §3.6](11-cloud-readiness-assessment.md)), the trigger becomes an ordinary submit button styled as
a link, and the `javascript:` URL is removed.

### 6.3 Session and cart identity

The cart key lives in session, keyed by a constant
[src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:19], and is reached two different ways: through
`HttpContextBase.Session` inside `GetCartId` [:161-180] — read at [:163], written at [:167] for a
signed-in user and at [:175] for an anonymous one, read again at [:179] — and through the MVC
`Controller.Session` property [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:39].

**Session in the target is opt-in and in-memory by default.** So two registrations are required
rather than optional: `AddSession` over a **distributed cache**, and `UseSession` in the pipeline
(section [2.4](#24-what-programcs-contains-in-order)). Omitting them does not fail to compile — it
produces a cart that empties itself unpredictably.

Three specifics:

- **The cache is SQL-backed**, sharing the one database of section
  [4.5](#45-two-contexts-two-migration-sets-two-history-tables). Its **table is provisioned by
  [06](06-azure-hosting-recommendations.md)** and created **first** in the ordering of section
  [5.3](#53-who-applies-ddl-and-in-what-order).
- **The session key constant is preserved** and both access paths converge on one: the service of
  section [4.8](#48-the-service-layer-and-the-two-patterns-deliberately-not-adopted) owns cart-id
  resolution, and `AccountController` stops writing session directly — it calls the service, which is
  also what makes the cross-store sequence of section [4.3](#43-the-cross-store-consistency-model)
  expressible in one place.
- **Session cookie attributes are set explicitly**, on the same principle as section
  [6.1](#61-the-policy-table-every-row-labelled): secure, `HttpOnly`, `SameSite=Lax`, and marked
  essential so that it is not suppressed by consent gating.

[11 §3.1](11-cloud-readiness-assessment.md) owns the statefulness assessment, including the precise
point that instance affinity permits multiple instances **today** while foreclosing *resilient*
scale-out. This document's contribution is the mechanism: distributed session plus the persisted key
ring of section [6.1](#61-the-policy-table-every-row-labelled), after which affinity is no longer
load-bearing.

---

## 7. Anti-forgery — three separate problems

### 7.1 The policy

**Global automatic validation.** `AutoValidateAntiforgeryTokenAttribute` is registered as a global
filter, so **every non-`GET` request is validated by default**, with opt-outs that are explicit,
individually justified and reviewable. This is the slot vacated by
`FilterConfig.RegisterGlobalFilters` (section
[2.2](#22-the-five-application_start-registrations-do-not-share-one-fate)).

Opt-in-per-action is rejected for a reason visible in the repository: the source uses opt-in, and it
covers **one controller of the four that need it**. A policy whose failure mode is "someone forgot an
attribute" has already failed here once.

**The emission/validation asymmetry is why the filter is required and not merely tidy.**
`@Html.AntiForgeryToken()` appears **10** times across MVC 5's views
(`git grep -o -E '@Html\.AntiForgeryToken\(\)' -- 'src/MVC5/*.cshtml' | wc -l` → `10`). In the target,
**emission becomes automatic** — form tag helpers emit the field without a helper call, so all ten
calls simply disappear. **Validation does not become automatic.** Removing the ten emission calls while
assuming validation came along is the exact shape of a silent regression: every form still carries a
token and nothing checks it.

Adopting the global policy exposes three problems a namespace-level port would miss. Each gets a named
resolution.

### 7.2 Problem one — coverage today is partial

There are **13** POST actions in the migration source and **8** of them validate.

| Controller | POST actions | Validated |
| --- | --- | --- |
| `AccountController` | 8 — attributes at [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:53], [:86], [:112], [:146], [:197], [:235], [:262], [:300] | **8** — `[ValidateAntiForgeryToken]` at [:55], [:88], [:113], [:147], [:199], [:236], [:264], [:301] |
| `StoreManagerController` | **3** — [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:53] Create, [:86] Edit, [:116] `DeleteConfirmed` | **0** |
| `ShoppingCartController` | 1 — [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:54] `RemoveFromCart` | **0** |
| `CheckoutController` | 1 — [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:25] `AddressAndPayment` | **0** |

**Five state-changing POST actions are unprotected**: three administration writes, cart removal, and
the checkout write — the one that creates an order.

**A census trap, stated because getting it wrong understates the work.** `grep -c '\[HttpPost\]'`
against `StoreManagerController.cs` returns **2**, not 3, because its third POST is declared
`[HttpPost, ActionName("Delete")]` at [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:116]
— a combined attribute list the literal pattern misses.
**The real count is 3.** The correct census matches on the attribute *prefix*, and it must also
tolerate the UTF-8 byte-order mark that all 27 source files carry (section
[9.1](#91-the-census-and-how-to-count-it)). The reproducing commands for both the naive and the correct
form are in [Appendix A](#appendix-a--reproducibility).

**Resolution: the global filter covers all five, plus every future non-`GET` endpoint, with no
per-action attribute.** Adopting it needs no code change at four of the five sites — only at the
fifth, which is the AJAX post of section [7.4](#74-problem-three--cart-removal-posts-without-a-form).
The eight existing `[ValidateAntiForgeryToken]` attributes are removed as redundant, which is safe
**only because** the global filter is registered; the two changes are one change and must not be split
across commits.

### 7.3 Problem two — one state-changing action is a `GET`, which no policy can cover

`AddToCart` is declared with **no verb attribute**
[src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:33] and it **mutates**: it loads the
album [:37-38], resolves the cart [:41], adds the item [:43], calls `SaveChanges` [:45] and redirects
[:48]. The comment above it even documents it as `GET: /ShoppingCart/AddToCart/5` [:31].

**No anti-forgery policy can protect a state-changing `GET`**, because a `GET` is issued by any image
tag, link or prefetch without a token and without the user's intent. This is not a coverage gap to
close with a filter; it is a verb that is wrong. [09 §6.1](09-security-assessment.md) owns it as a
finding across all three editions.

**Resolution: convert it to a token-protected `POST`.**

- The action gains an explicit `[HttpPost]`, and the global filter of section [7.1](#71-the-policy)
  validates it with no further attribute.
- **Its call site changes from a link to a posting form.**
  `@Html.ActionLink("Add to cart", "AddToCart", "ShoppingCart", new { id = Model.AlbumId }, "")`
  [src/MVC5/MvcMusicStore/Views/Store/Details.cshtml:28-29] becomes a form posting to the same path,
  with the album id as a route value and a submit control styled to match today's appearance. The
  surrounding `<p class="btn btn-default">` [:26] is restructured accordingly — a `btn` class on a
  paragraph wrapping a link is Bootstrap 3-era markup that section
  [8.5](#85-the-bootstrap-upgrade-is-markup-work-not-a-version-bump) revisits anyway.
- **The response behaviour is preserved**: a redirect to the cart index
  [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:48], so the post-redirect-get
  pattern the page already follows is unchanged.

**This is a deliberate, approval-owned interface delta. The path is unchanged; the verb is not.** Any
external bookmark, link or automation targeting `/ShoppingCart/AddToCart/5` **stops working** — it
will render the cart page or a 405 rather than adding an item. There is no way to have both: a
bookmarkable URL that adds to a cart *is* the vulnerability. It is listed among the approved deltas of
section [11.5](#115-the-full-set-of-approved-deltas) with **Security** as its approval owner, and
section [12.4](#124-required-coverage) requires the new verb to be tested and the old one to be
asserted **not** to mutate.

### 7.4 Problem three — cart removal posts without a form

The cart page removes an item with
`$.post(PostToUrl, { "id": recordToDelete }, ...)`
[src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:17]. The trigger is a plain anchor carrying
`data-id` and `data-url` attributes [:74-75], read at [:12-13]. **There is no surrounding form.**

So the global policy of section [7.1](#71-the-policy) would **reject every cart removal**: no form
means no tag helper, no tag helper means no emitted token, and no token means a rejected POST. This is
the one site where adopting the policy requires client-side work, and it is the reason the policy has
to be adopted deliberately rather than switched on.

**The transport is form-urlencoded, not JSON.** No `contentType` and no `JSON.stringify` is specified,
so jQuery serializes the object as a form body — jQuery's default. That matters because it means a
hidden form field **would** work; the transport does not force the answer, so the answer is a choice
and is recorded as one.

**The selected contract is the `RequestVerificationToken` request header.**

- **Server side:** `AntiforgeryOptions.HeaderName` is set to `RequestVerificationToken` in
  `Program.cs` (section [2.4](#24-what-programcs-contains-in-order)). The name is stated once, in
  configuration, rather than assumed to be a default.
- **Client side:** the cart page renders a token into the document — a hidden field emitted by an
  otherwise-empty form, or an equivalent single emission point — and the script reads its value and
  sends it as the `RequestVerificationToken` header on the post. The value is read at post time rather
  than cached at page load, so a token refreshed by any other interaction on the page is still correct.
- **Why the header rather than a body field:** it keeps the token out of the request body, so it
  applies uniformly to **any** future scripted endpoint regardless of that endpoint's content type. A
  body field only works for form-urlencoded and multipart bodies, and would have to be reinvented the
  first time an endpoint posts JSON.

**Three tests, not one** (section [12.4](#124-required-coverage)): a **valid** token removes the item
and returns the expected payload; a **missing** token is rejected and the cart is **unchanged**; an
**invalid** token is rejected and the cart is **unchanged**. The last two must assert on the data, not
only on the status code — a rejection that still mutated would pass a status-only assertion.

The response side of this same endpoint has its own, independent problem; it is section
[8.7](#87-the-json-contract--annotate-one-model-not-the-policy).

---

## 8. Views, static assets and the wire contracts

### 8.1 Bundling and static assets — no bundler

Five bundles are registered [src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs:11], [:14], [:19],
[:22], [:26], and two token forms in them are the real obstacle rather than the bundling: a
**`{version}` token** — `"~/Scripts/jquery-{version}.js"` [:12] — and **glob tokens** —
`"~/Scripts/jquery.validate*"` [:15] and `"~/Scripts/modernizr-*"` [:20]. No successor supports either
form ([12 F-12-02](12-migration-blockers.md)).

**Resolution: no bundler at all.**

- The 27 files under `Content/`, `Scripts/`, `Images/` and `fonts/`
  (`git ls-files 'src/MVC5/MvcMusicStore/Content/*' 'src/MVC5/MvcMusicStore/Scripts/*' 'src/MVC5/MvcMusicStore/Images/*' 'src/MVC5/MvcMusicStore/fonts/*' | wc -l` → `27`) plus the
  web-root `favicon.ico` move to **`wwwroot`**, served by **static-file middleware** (section
  [2.4](#24-what-programcs-contains-in-order)).
- Cache busting uses the framework's **version-appending tag helper** on `<script>` and `<link>`
  elements, which appends a content hash. That replaces the `{version}` token's purpose — the token
  resolved a *filename* version, the tag helper appends a *content* version, which is strictly better
  for cache correctness and needs no filename convention.
- **The reason a bundler is not reintroduced:** minification and concatenation would require a
  JavaScript toolchain — a package manifest, a lock file, a build step and a CI dependency — for
  **27 asset files** on an application that has no build automation of any kind today
  ([08 §7.2](08-technical-debt-register.md)). That is a permanent build dependency bought for a
  one-time size saving on a catalog application. It is not justified, and if it ever becomes justified
  it is an additive decision taken then.
- **All 11 helper call sites are rewritten**, because both helpers cease to exist along with the five
  bundle virtual paths they name: **10** `@Scripts.Render` and **1** `@Styles.Render`
  (`git grep -o '@Scripts.Render' -- 'src/MVC5/*.cshtml' | wc -l` → `10`; the same for
  `@Styles.Render` → `1`). Three are in the shared layout — `@Styles.Render("~/Content/css")`
  [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:7], `@Scripts.Render("~/bundles/modernizr")`
  [:8], `@Scripts.Render("~/bundles/jquery")` [:41] and `@Scripts.Render("~/bundles/bootstrap")`
  [:42] — and the remainder sit in the Account, Checkout and StoreManager views, each inside a
  `@section Scripts` block, for example [src/MVC5/MvcMusicStore/Views/Account/Manage.cshtml:28]. Each
  becomes a direct `<script>` or `<link>` element with the tag helper.
- **Two bundle members do not come across at all**, because the libraries they name are removed rather
  than updated: `respond.js` [src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs:24] and the modernizr
  bundle [:19-20]. Both exist for browsers the target does not support. [04 §9.2](04-dotnet8-migration-strategy.md)
  owns their removal and [06](06-azure-hosting-recommendations.md) states the browser matrix; section
  [11.5](#115-the-full-set-of-approved-deltas) lists the narrowed matrix as an approved delta.
- **The casing is corrected in the move.** The style bundle registers `"~/Content/site.css"`
  [src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs:28] while the tracked file is
  `src/MVC5/MvcMusicStore/Content/Site.css` — verified: the directory holds `Site.css`,
  `bootstrap.css` and `bootstrap.min.css`, and the registration's lowercase `s` matches none of them.
  IIS resolves this; a case-sensitive filesystem does not, and the stylesheet 404s with no error
  anywhere. Every asset reference is therefore emitted against the **actual** filename after the move.
  [11 §3.7](11-cloud-readiness-assessment.md) owns the finding and
  [06](06-azure-hosting-recommendations.md) owns the audit that gates its hosting recommendation.

Client-side library pins and the acquisition mechanism, including `libman.json`, are
[04 §9](04-dotnet8-migration-strategy.md)'s. This document names no version.

### 8.2 Child actions — three view components

Child actions do not exist in the target. MVC 5 declares `[ChildActionOnly]` on three actions, and
each becomes a **view component** with its call site rewritten.

| Source action | Renders | Target component | Call site to rewrite |
| --- | --- | --- | --- |
| `StoreController.GenreMenu` [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:43-55] | The genre navigation in the shared layout | `GenreMenuViewComponent` + `Views/Shared/Components/GenreMenu/Default.cshtml` | `@Html.Action("GenreMenu", "Store")` [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:25] |
| `ShoppingCartController.CartSummary` [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:86-99] | The cart count in the same layout | `CartSummaryViewComponent` + `Views/Shared/Components/CartSummary/Default.cshtml` | `@Html.Action("CartSummary", "ShoppingCart")` [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:26] |
| `AccountController.RemoveAccountList` [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:316-322] | The external-login removal list | **Not written** — see below | `@Html.Action("RemoveAccountList")` [src/MVC5/MvcMusicStore/Views/Account/Manage.cshtml:22] |

Three specifics:

- **`ViewBag` does not cross the boundary.** Both surviving components pass data through `ViewBag`
  today — `ViewBag.CartCount` and `ViewBag.CartSummary`
  [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:95-96], read by the partial at
  [src/MVC5/MvcMusicStore/Views/ShoppingCart/CartSummary.cshtml:1], [:4], [:6], and
  `ViewBag.ShowRemoveButton` [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:320]. A view
  component has its **own** view context, so each gains a **typed view model** instead. This is not
  optional polish: `ViewBag` values set in a component are not visible to the parent view and vice
  versa, so a copy-paste port renders an empty cart badge on every page and nothing says why.
- **They execute in a different scope and must be re-verified.** A child action ran as a nested
  action invocation; a view component executes inline during the parent's render, inside the same
  request scope and therefore against the **same** scoped `DbContext` as the parent view's controller
  (section [4.2](#42-what-scoped-registration-actually-changes--and-what-it-does-not)). That is
  usually benign and occasionally not — a component reading an entity the parent has modified but not
  saved now sees the modification. Both surviving components are read-only, so the required work is a
  **test that each renders correctly on a page whose action has already written to the context**,
  which for the cart summary means the page reached immediately after an add-to-cart.
- **`RemoveAccountList` is not ported.** Its component, its view and the action itself are **deleted**
  along with the rest of the external-login surface (section
  [8.3](#83-the-five-views-that-name-legacy-types)), and the `@Html.Action` call at
  [src/MVC5/MvcMusicStore/Views/Account/Manage.cshtml:22] is removed rather than rewritten. The
  `<section id="externalLogins">` block that contains it [:21-24] goes with it.

### 8.3 The five views that name legacy types

**A plan that treats all 29 views as a bulk copy breaks in exactly these five places.**
(`git ls-files 'src/MVC5/*.cshtml' | wc -l` → `29`.) Each is listed with its line-level problem and its
resolution.

**(1) `Views/Shared/Error.cshtml` — a removed model type, and the application's first real error
policy.**

`@model System.Web.Mvc.HandleErrorInfo` [src/MVC5/MvcMusicStore/Views/Shared/Error.cshtml:1]. The type
does not exist in the target and the exception-handling middleware supplies **no Razor equivalent**, so
there is nothing to substitute into the directive. The rest of the file is two static headings [:7-8]
and it renders **no property of the model at all**.

**Resolution: define `ErrorViewModel.cs` and specify the whole policy, because the source has none to
preserve.** `HandleErrorAttribute` [src/MVC5/MvcMusicStore/App_Start/FilterConfig.cs:10] only engages
when custom errors are enabled, and **no edition enables them** — there is no live `<customErrors>`
element in any configuration file in the repository (verified; command in
[Appendix A](#appendix-a--reproducibility); [09 §6.10](09-security-assessment.md) owns the finding). So the
attribute is closer to inert than active as shipped, and **the target's error behaviour is being
chosen, not preserved.** It is therefore stated in full:

- **The error route.** The exception-handler middleware forwards to `/Home/Error`, which renders this
  view. Status-code responses that reach the pipeline without a body — a 404 from the three
  `NotFound()` results of section [13](#13-resolution-register--all-22-blockers-of-deliverable-12), or
  a 405 from a wrong verb — are re-executed into the same route with the status code preserved.
- **Unhandled exceptions** produce **500** and the generic view. **Status-code responses** preserve
  their own code — a 404 stays a 404, so a crawler or a monitor is not told an error page is a success.
  Neither path returns 200.
- **What is logged.** The exception, its stack trace, the request path and method, the trace
  identifier, and the authenticated user name where present — written through `ILogger` at `Error`
  level. **The trace identifier is also rendered into the view**, so a user can quote a value that
  correlates to the log entry. (The collection mechanism is
  [06](06-azure-hosting-recommendations.md)'s and is not restated here.)
- **What the response may disclose.** The two generic headings [:7-8], preserved verbatim, plus the
  trace identifier. **No exception type, no message, no stack trace, no query text and no
  configuration value** in any environment other than Development, where the developer exception page
  handles it instead (section [2.4](#24-what-programcs-contains-in-order)).
- **The view's own user-facing text is preserved.** Only its model and its plumbing change.

**Do not attribute MVC 4's exception disclosure to this view.** MVC 4's disclosure is in its
*controller* — `ModelState.AddModelError("", e)` passing the exception object rather than a string
[src/MVC4/MvcMusicStore/Controllers/AccountController.cs:211-213], which
[09 §4.9](09-security-assessment.md) owns — and MVC 5's error view discloses nothing at all. This view
is rewritten for exactly one reason: its model type is gone.

**(2) `Views/Shared/_LoginPartial.cshtml` — two separate problems in one file.**

- `Request.IsAuthenticated` [src/MVC5/MvcMusicStore/Views/Shared/_LoginPartial.cshtml:2] does not exist
  on the target's request type. **Resolution:** an explicit, null-handled identity check —
  `User.Identity?.IsAuthenticated == true` — because both `User` and `Identity` are reference-typed in
  the target and an unauthenticated request can present a non-null principal with a null name.
- `User.Identity.GetUserName()` [:10] is an extension method from the Identity 1.0 namespace imported
  at [:1]. **Resolution:** the namespace import is updated and the call becomes a direct read of the
  identity name, which is what the extension returned. The rendered text — `"Hello " + name + "!"` —
  is preserved exactly.

**Both signed-in and signed-out rendering need tests**, because the file's **entire structure** is a
conditional on the first of those two problems [:2-22]: the signed-in branch renders the sign-out form
and the manage link [:4-14], the signed-out branch renders register and log-in links [:18-21]. A
null-handling mistake does not error — it silently renders the wrong branch, and a suite that only
exercises one of them cannot tell.

**(3) `Views/Account/Manage.cshtml` — namespace import plus a removed call site.**
`@using Microsoft.AspNet.Identity;` [src/MVC5/MvcMusicStore/Views/Account/Manage.cshtml:2] becomes the
target's Identity namespace. Its `@Html.Action("RemoveAccountList")` [:22] and the surrounding
external-logins block [:21-24] are **removed** per section
[8.2](#82-child-actions--three-view-components), and its `@Scripts.Render` [:28] is rewritten per
section [8.1](#81-bundling-and-static-assets--no-bundler).

**(4) `Views/Account/_ChangePasswordPartial.cshtml` — namespace import plus an identity read.**
`@using Microsoft.AspNet.Identity` [src/MVC5/MvcMusicStore/Views/Account/_ChangePasswordPartial.cshtml:1]
becomes the target's Identity namespace, and `@User.Identity.GetUserName()` [:4] becomes a direct
identity-name read, as in (2). Its `@Html.AntiForgeryToken()` [:8] disappears into automatic emission
(section [7.1](#71-the-policy)).

**(5) `Views/Account/_RemoveAccountPartial.cshtml` — and `UserLoginInfo` is *not* renamed.**
`@model ICollection<Microsoft.AspNet.Identity.UserLoginInfo>`
[src/MVC5/MvcMusicStore/Views/Account/_RemoveAccountPartial.cshtml:1]. **The type name survives** — the
target's Identity keeps `UserLoginInfo` under its own namespace — so this looks like a one-word
namespace edit and is not: **the type's shape and the API that produces it both differ**, so the change
would be a namespace update **plus** adaptation of how the collection is obtained (today
`UserManager.GetLogins(User.Identity.GetUserId())`
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:319]) and of what the view reads from each
element (`account.LoginProvider` [:11], `account.ProviderKey` [:20]).

**In this port the question is moot, because the view is deleted.**

**The external-login decision, made and recorded here.** [12 §7.3](12-migration-blockers.md) records
this as 05's choice: map `ChallengeResult` onto the framework's external-challenge flow, or **remove
the disabled external-login surface entirely**. **It is removed.** Reasons:

- **Nothing is reachable today.** All four provider registrations are commented out
  [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:23-35], with empty credentials in the commented
  arguments [:24-25], [:28-29], [:32-33]. No user has ever signed in through this surface, so **no
  user-visible capability is lost.**
- **It is deployed attack surface**, which [09 §6.11](09-security-assessment.md) records and
  [08 §9.3](08-technical-debt-register.md) records as dead scaffolding.
- **Mapping it means writing and testing a flow nobody uses**, including a challenge, a callback, a
  confirmation view, an association flow and a disassociation flow.

**What removal deletes**, so that the scope is explicit and nothing is left half-removed: the
`ChallengeResult` nested class [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:394-420] and its
`ExecuteResult` override [:411]; the `IAuthenticationManager` property [:338-344] and the `XsrfKey`
constant [:336]; `RemoveAccountList` [:316-322]; the external-login, callback, association,
disassociation and external-login-failure actions and their view models; the external sign-in cookie
registration [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:20] and the four commented
registrations [:23-35]; and **five** of the nine views under `Views/Account/` —
`_RemoveAccountPartial.cshtml`, `_ExternalLoginsListPartial.cshtml`, `ExternalLoginFailure.cshtml`,
`ExternalLoginConfirmation.cshtml` and `_SetPasswordPartial.cshtml` — along with the
`<section id="externalLogins">` block of `Manage.cshtml` that hosts two of them
[src/MVC5/MvcMusicStore/Views/Account/Manage.cshtml:21-24].

`_SetPasswordPartial.cshtml` is in that list for a second-order reason worth stating, because it is
the one deletion that is not obviously part of the surface: it exists **only** for accounts with no
local password, and only an external login can create such an account. With the surface removed it is
unreachable, so the `ViewBag.HasLocalPassword` conditional
[src/MVC5/MvcMusicStore/Views/Account/Manage.cshtml:12-19] collapses to the change-password partial
[:14] and the set-password branch [:18] is deleted with it. **Verify against the migrated data before
deleting**: if any migrated account has a null `PasswordHash`, that account has no local password and
the branch is reachable after all — in which case the surface removal is unchanged but those accounts
need a password-reset path instead, which is the same fallback section
[5.5](#55-the-identity-transition--a-schema-decision-plus-a-data-migration) already defines for hash
verification failure.

**Re-adding external sign-in later is an additive, scoped decision** with its own provider selection,
its own pinned packages and its own tests. This document does not pre-build it, and section
[6.1](#61-the-policy-table-every-row-labelled) labels the removal a deliberate change. **Route and
behaviour tests attach either way**: the removed routes must return 404, and the sign-in, register,
sign-out and manage flows must be unaffected.

### 8.4 The remaining views

Twenty-nine views exist in the migration source; **five are deleted** by section
[8.3](#83-the-five-views-that-name-legacy-types)'s external-login decision, **one becomes a view
component's `Default.cshtml`** (`ShoppingCart/CartSummary.cshtml`, section
[8.2](#82-child-actions--three-view-components)), and **four need the per-line work** listed in section
[8.3](#83-the-five-views-that-name-legacy-types). That leaves **nineteen** that port as Razor with
mechanical changes only.

Those nineteen — and the four per-line ones, once their type problems are resolved — need four
mechanical changes and have no type problems:
`@Html.Partial` becomes the `<partial>` tag helper at **5** call sites
(`git grep -o '@Html.Partial' -- 'src/MVC5/*.cshtml' | wc -l` → `5`), `@Url.Content` remains valid at
its **4** call sites (→ `4`), `@Html.AntiForgeryToken()` disappears into automatic emission at its 10
call sites (section [7.1](#71-the-policy)), and the bundling helpers are rewritten per section
[8.1](#81-bundling-and-static-assets--no-bundler). Form and input helpers are source-compatible; tag
helpers are the target convention and are adopted where a form is being touched anyway — which,
because of the anti-forgery change, is every form.

**One asymmetry makes this less safe than it sounds, and it cuts in the port's favour.** These 29 views
have **never been compile-checked**: `MvcBuildViews` is `false`, so no view is compiled at build time
today ([10 §12.3](10-build-and-deployment-requirements.md), [08 §8.1](08-technical-debt-register.md)).
The target compiles Razor views as part of the build. So type errors that are invisible today —
including the removed model type in (1) above — become **build** errors, discovered for free rather
than at first request. The consequence for this plan is that the view port needs no separate
type-checking pass; it needs a build.

### 8.5 The Bootstrap upgrade is markup work, not a version bump

[04 §9.2](04-dotnet8-migration-strategy.md) pins the target Bootstrap version. **The markup work is
this document's**, and it is not optional: **upgrading the package without touching the markup changes
the rendered page.**

Bootstrap 3 navbar classes are pervasive in the shared layout, and several were renamed or removed
after Bootstrap 3:

| Class | Location |
| --- | --- |
| `navbar navbar-inverse navbar-fixed-top` | [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:12] |
| `navbar-header` | [:14] |
| `navbar-toggle` (with three `icon-bar` spans at [:16], [:17], [:18]) | [:15] |
| `navbar-brand` | [:20] |
| `navbar-collapse collapse` | [:22] |
| `nav navbar-nav` | [:23] |
| `navbar navbar-fixed-bottom navbar-default text-center` | [:35] |
| `nav navbar-nav navbar-right` | [src/MVC5/MvcMusicStore/Views/Shared/_LoginPartial.cshtml:8], [:18] |
| `form-horizontal`, `col-md-*`, `control-label`, `form-group`, `btn btn-default` | [src/MVC5/MvcMusicStore/Views/Account/_ChangePasswordPartial.cshtml:6], [:12-35] and the other form views |
| `table`, `table-striped` | [src/MVC5/MvcMusicStore/Views/Account/_RemoveAccountPartial.cshtml:6], [src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:47] |

**Resolution: a visual-preservation port.** Every affected class name is mapped old-to-new, the
collapse mechanism's data attributes are updated to the target library's form, the horizontal-form
grid is re-expressed in the target's grid utilities, and **acceptance is defined as the rendered layout
matching the Bootstrap 3 baseline captured *before* the port.** The baseline capture and the review are
scoped in section [12.5](#125-rendered-appearance-comparison-is-separate-and-manual); they are
deliberately **not** claimed by the automated suite.

**Glyphicons are removed rather than replaced with an icon library.** They were dropped from Bootstrap
after 3. Use is decorative and confined to **exactly two views** — verified:

```text
src/MVC5/MvcMusicStore/Views/ShoppingCart/CartSummary.cshtml:5:  <span class="glyphicon glyphicon glyphicon-shopping-cart"></span>
src/MVC5/MvcMusicStore/Views/Store/Details.cshtml:27:            <span class="glyphicon glyphicon-shopping-cart"></span>
```

**Neither is the shared layout** — a natural assumption, since the cart badge renders *into* the
layout, but the markup lives in the cart-summary partial rather than in `_Layout.cshtml`. The two spans
are removed; the adjacent text and link in each view are preserved, so both controls keep their label
and their behaviour and lose only the pictogram. (The cart-summary span also carries the `glyphicon`
class twice [:5], which is worth nothing except as a sign that nobody has looked at this markup in a
long time.)

**Removing them adds no dependency, no acquisition mechanism and no licence question**, which is
precisely the argument for removal over substitution at this scale. It is an **approved visual delta**
(section [11.5](#115-the-full-set-of-approved-deltas)), not an unnoticed regression. **If a stakeholder
wants icons retained, that is a scoped decision with its own pinned source and its own licence
review — this document does not assume one.**

### 8.6 View serving and `_ViewImports`

`Views/Web.config` carries two unrelated constructs and they have opposite fates.

**The `BlockViewHandler` mapping ends, and needs no replacement.**
`<remove name="BlockViewHandler"/>` followed by
`<add name="BlockViewHandler" path="*" verb="*" preCondition="integratedMode" type="System.Web.HttpNotFoundHandler" />`
[src/MVC5/MvcMusicStore/Views/Web.config:31-32] exists to stop `.cshtml` files under `Views/` being
requested directly. **The target does not serve content-root `.cshtml` files at all**, so the behaviour
the handler enforces is the default rather than something to configure. `preCondition="integratedMode"`
is meaningless outside IIS in any case.

**The namespace registration has a direct successor: `Views/_ViewImports.cshtml`.** Six namespaces are
imported into every view today [src/MVC5/MvcMusicStore/Views/Web.config:14-21]. **Three survive and
three are dead:**

| Namespace | Line | Target |
| --- | --- | --- |
| `System.Web.Mvc` | [:15] | **Survives, remapped** to the target's MVC namespace |
| `System.Web.Mvc.Ajax` | [:16] | **Dead.** The unobtrusive-Ajax helper family does not exist in the target, and **no view uses it** — there is no `@Ajax.` call anywhere in MVC 5's views |
| `System.Web.Mvc.Html` | [:17] | **Dead** as a namespace. The `Html` helpers it contained are reached through the view's own `Html` property in the target and need no import |
| `System.Web.Optimization` | [:18] | **Dead entirely** — the package has no successor (section [8.1](#81-bundling-and-static-assets--no-bundler)) |
| `System.Web.Routing` | [:19] | **Survives, remapped** to the target's routing namespace — retained only if a ported view actually references a routing type; **verify during the port and drop it if none does**, rather than carrying an unused import forward |
| `MvcMusicStore` | [:20] | **Survives** as the application root namespace, and `MvcMusicStore.Models` is added alongside it, since every strongly-typed view names a model under it — for example [src/MVC5/MvcMusicStore/Views/Store/Details.cshtml:1] and [src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:1] |

`_ViewImports.cshtml` additionally registers the framework's tag helpers, which the source has no
equivalent of and which sections [7.1](#71-the-policy) and
[8.1](#81-bundling-and-static-assets--no-bundler) both depend on. The Razor host and page-base-type
registration [:12-13] and the `configSections` group that declares them [:4-9] **end**: the
configuration section does not exist in the target and the base type is a framework concern rather
than an application one. `Views/_ViewStart.cshtml` ports unchanged.

The redundant `webpages:Enabled` key [:26] ends with the rest of the Web Pages surface (section
[3.1](#31-configuration-webconfig-becomes-appsettingsjson-read-through-iconfiguration)).

### 8.7 The JSON contract — annotate one model, not the policy

The cart-removal endpoint returns a view model with five **PascalCase** properties — `Message`,
`CartTotal`, `CartCount`, `ItemCount`, `DeleteId`
[src/MVC5/MvcMusicStore/ViewModels/ShoppingCartRemoveViewModel.cs:5-9] — populated at
[src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:73-81] and returned as
`return Json(results)` [:83].

The page's JavaScript reads **exactly those PascalCase names**:
`data.ItemCount` [src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:21], `data.DeleteId` [:22],
`data.ItemCount` again [:24], `data.CartTotal` [:27], `data.Message` [:28] and `data.CartCount` [:29].

**The target's default web serializer camel-cases property names, and the failure is silent.** After
the port the request still succeeds, the response is still 200, the JSON is still well-formed — and
every one of those reads evaluates to `undefined`. The row does not fade out [:22], the count does not
update [:24], the total does not change [:27], the message stays empty [:28] and the header badge stops
tracking [:29]. **No error occurs, so nothing is logged.**

**Get the attribution right, because the wrong attribution produces the wrong fix.** The serializer
producing PascalCase today is `JavaScriptSerializer`, by way of MVC 5's `JsonResult` — **not**
Newtonsoft.Json, which is pinned as a package but never called from application source.
[12 F-12-16](12-migration-blockers.md) establishes both facts with its command; this document does not
re-derive them. The consequence is that **removing Newtonsoft.Json is the removal of an unused
dependency, not a serializer replacement** ([04 §8.4](04-dotnet8-migration-strategy.md) owns that
outcome).

**Resolution: annotate that one response model with explicit JSON property names. Do not change the
application-wide serializer policy.** [12 §7.3](12-migration-blockers.md) records the choice as 05's;
it is taken this way for three reasons:

1. **The affected surface is exactly one endpoint and one model.** There is one `Json(...)` return in
   the entire migration source, at [:83].
2. **A global PascalCase policy would be a decision about every future endpoint**, taken to
   accommodate one 2013-era script — precisely the kind of application-wide default that section
   [6](#6-authentication-policy-is-decided-not-inherited) argues against inheriting silently.
3. **The annotation is local, visible and self-documenting**: a reader of the model sees why the names
   are pinned, whereas a serializer option in `Program.cs` is invisible from the model and from the
   script.

The alternative of changing the JavaScript to read camelCase was considered and **not** taken: it
would work, but it changes the client and the server contract at once for no gain, and the annotation
keeps the wire format byte-identical to the baseline — which is what lets the same assertion run
against both runtimes in section [12](#12-the-test-suite-architecture-and-coverage).

**The test asserts on property names and values, not merely on a 200** (section
[12.4](#124-required-coverage)) — a status-only assertion is exactly what this failure passes.

### 8.8 The checkout input model — ten properties, not nine

`TryUpdateModel(order)` [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:29] is applied to an
`Order` constructed one line earlier [:28], inside a POST action whose parameter is a raw
`FormCollection` [:26]. What makes the call safe today is a **separate** construct: a class-level
attribute on the entity,
`[Bind(Include = "FirstName,LastName,Address,City,State,PostalCode,Country,Phone,Email")]`
[src/MVC5/MvcMusicStore/Models/Order.cs:8]. That include list is the **entire** over-posting control at
checkout ([09 §6.4](09-security-assessment.md) owns the consequence), and both halves have to move
together.

**Both halves fail to compile in the target.** Only the asynchronous overload,
`TryUpdateModelAsync`, exists — so [:29] does not compile — and `[Bind(Include = ...)]` is superseded
by explicit binding models, so the attribute at [Order.cs:8] has no target either.

**Resolution: `Binding/CheckoutInputModel.cs`, carrying TEN properties.** Nine is the number written
down in the repository and **ten is the number the action actually reads**:

- **Nine** from the `[Bind]` include list [src/MVC5/MvcMusicStore/Models/Order.cs:8]: `FirstName`,
  `LastName`, `Address`, `City`, `State`, `PostalCode`, `Country`, `Phone`, `Email`. Each carries its
  existing DataAnnotations attributes across verbatim — `[Required]`, `[StringLength]`,
  `[DisplayName]`, `[DataType]` and the email `[RegularExpression]` [:20-61] — so client and server
  validation messages are preserved.
- **The tenth is `PromoCode`**, which the action reads **separately, straight out of the raw form**:
  `values["PromoCode"]` [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:33], compared
  case-insensitively [:33-34] against `const string PromoCode = "FREE"` [:12]. It is not on the
  `[Bind]` list because it is not bound at all.

**`PromoCode` belongs on the input model and NOT on the `Order` entity.** `Order` has no such property
and must not gain one: its surface is fourteen properties — the nine bound ones, four suppressed with
`[ScaffoldColumn(false)]` (`OrderId` [:11-12], `OrderDate` [:14-15], `Username` [:17-18], `Total`
[:63-64]) and the `OrderDetails` navigation [:66]. Adding a tenth **persisted** property would store a
transient form value as a column, and would appear in the schema diff of section
[5.1](#51-domain-data-migration-starts-with-schema-extraction) as an unexplained difference.

**Why a nine-property model is the dangerous near-miss.** It compiles, binds and persists correctly —
and **silently loses promo-code handling**. The order-completion branch [:38-55] is guarded by the
promo-code comparison [:33-34], so with the value never bound the comparison always fails and **every
checkout returns to the form** at [:36] with no error message and no log entry. **Test valid, invalid
and missing promo-code values** (section [12.4](#124-required-coverage)).

Two further changes in the same action:

- **`Order.cs` loses `using System.Web.Mvc`** [src/MVC5/MvcMusicStore/Models/Order.cs:4], which exists
  purely to reference that attribute, along with the attribute itself. A model-layer file with an MVC
  dependency is why this port is not confined to `Controllers/`.
- **The bare `catch` is replaced.** `catch { return View(order); }`
  [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:58-62] has **no exception variable**: it
  discards the exception, redisplays the view and records nothing, wrapping the entire order
  transaction [:31-62] ([08 §5.7](08-technical-debt-register.md) owns it as debt). The port catches
  specific exceptions, **logs through `ILogger`** with the order context and the trace identifier, adds
  a user-facing model error rather than silently redisplaying an empty form, and lets anything
  unanticipated reach the exception middleware of section
  [8.3](#83-the-five-views-that-name-legacy-types). (The collection mechanism is
  [06](06-azure-hosting-recommendations.md)'s.)

**The checkout path is the one most exposed to the lifetime change, and must be re-verified
explicitly.** It adds the order, resolves the cart, writes order details, empties the cart and calls
`SaveChanges` in one sequence [:44-51] — and it already passes **its own** context into `GetCart` [:47],
so it is *not* an instance of the multi-context problem. What it is exposed to is the change from a
controller-lifetime context to a request-scoped one across a five-step write: the required verification
is that all of order, order details and cart-emptying commit together, and that a failure part-way
leaves **none** of them. The **account cart-migration path** [:30-40] must be re-verified for the same
reason, against the sequence of section [4.3](#43-the-cross-store-consistency-model).

---

## 9. A namespace substitution table is not a transformation plan

This section exists because writing a `FROM → TO` import table and calling it a migration plan is the
single most likely structural error in this port, and it is a tempting one: nineteen of twenty-seven
source files carry a legacy `using` directive, so a table looks like it covers the work. **It does not.
The imports are a symptom; the file's role in the target is the plan.**

### 9.1 The census, and how to count it

**19 of the 27 `.cs` files** in the migration source carry at least one directive naming a namespace
that does not survive the port.

**A methodology note that changes the answer.** All 27 files carry a **UTF-8 byte-order mark**, so a
naive `^\s*using` match against the file decoded as UTF-8 sees the BOM as a leading character on line 1
and **misses a first-line directive**. That count returns **17**. A BOM-tolerant per-file census
returns **19**. Both commands and both results are in
[Appendix A](#appendix-a--reproducibility) so the difference is checkable rather than asserted.

Per-directive counts, BOM-tolerant, counted as **files containing the directive**:

| Directive | Files |
| --- | --- |
| `using System.Web.Mvc;` | 11 |
| `using System.Web;` | 11 |
| `using Microsoft.Owin*` | 4 |
| `using Microsoft.AspNet.Identity*` | 4 |
| `using System.Data.Entity;` | 3 |
| `using Owin;` | 3 |
| `using System.Web.Routing;` | 2 |
| `using System.Web.Optimization;` | 2 |
| `using System.Configuration;` | 1 |

Two observations from that table rather than from the total. **`using System.Web.Mvc;` reaches the
model layer** — [src/MVC5/MvcMusicStore/Models/Order.cs:4] and
[src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:5] — so the port is not confined to `Controllers/`. And
**`using System.Configuration;`** appears exactly once, at
[src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:7], which is the file whose reads at [:23-24] are the
reason the options pattern is required (section
[3.1](#31-configuration-webconfig-becomes-appsettingsjson-read-through-iconfiguration)).

### 9.2 The four groups

Every file's disposition is set by its **role**, not by its imports, and the 27 files fall into four
groups that need four different kinds of work.

**Group A — six files whose imports are never rewritten, because the files are deleted.**
`Global.asax.cs`, `Startup.cs`, `App_Start/RouteConfig.cs`, `App_Start/FilterConfig.cs`,
`App_Start/Startup.App.cs` and `App_Start/Startup.Auth.cs`. Their `Owin`, `Microsoft.Owin.*`,
`System.Web.Routing`, `System.Web.Optimization` and `System.Configuration` directives **simply cease
to exist**; their responsibilities land in `Program.cs` per section [2](#2-the-composition-root). A
substitution table applied to these six files produces six rewritten files that should not exist.
`App_Start/BundleConfig.cs` is a seventh in the same position, and its responsibility ends rather than
moving (section [8.1](#81-bundling-and-static-assets--no-bundler)).

**Group B — three files that change identity, not just imports.**

- **`Models/Order.cs`** loses `using System.Web.Mvc` [:4] **and the class-level attribute it exists to
  support** [:8]. Removing the import without removing the attribute does not compile; removing both
  without writing the input model of section
  [8.8](#88-the-checkout-input-model--ten-properties-not-nine) removes the application's entire
  over-posting control at checkout.
- **`Models/ShoppingCart.cs`** loses its `System.Web` [:4] and `System.Web.Mvc` [:5] directives, loses
  the dead `Controller` overload [:29-32], **and stops being a model** — it becomes
  `Services/ShoppingCartService.cs` (section
  [4.8](#48-the-service-layer-and-the-two-patterns-deliberately-not-adopted)). Its file path, its
  namespace, its registration and its consumers all change.
- **`Controllers/AccountController.cs` is not import-rewritten either — it is rewritten wholesale**,
  `using Microsoft.Owin.Security;`
  [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:10] included, because
  `ChallengeResult` [:394] and its
  `ExecuteResult` override [:411] have no direct target and because its entire Identity surface changes
  shape. Section [8.3](#83-the-five-views-that-name-legacy-types) records the decision that removes
  the external-login half of it outright. This is the file [08 §4.2](08-technical-debt-register.md)
  sizes at 382 non-blank lines — roughly 18 percent of the migration source by the sizing metric — and
  the one part of the port with no line-for-line successor.

**Group C — the remaining ten files where a substitution table is broadly right.** The five other
controllers and the entity-and-context files: namespace substitution, plus the specific per-site work
sections [4](#4-dependency-injection-and-object-lifetime) and
[13](#13-resolution-register--all-22-blockers-of-deliverable-12) assign — constructor injection, the
three `NotFound()` conversions, eager loading, the `Dispose` deletions, and the explicit
`DbContextOptions` constructors. **This is the only group the table describes**, and it is ten files of
twenty-seven.

```text
FROM: using System.Web.Mvc;                 TO: the target's MVC namespace
FROM: using System.Data.Entity;             TO: the target's EF Core namespace
FROM: using Microsoft.AspNet.Identity[.*];  TO: the target's Identity namespaces
FROM: using System.Web;                     TO: (removed; HttpContextBase becomes the framework's context type)
FROM: using System.Web.Optimization;        TO: (no successor; the file is deleted)
FROM: using Owin; / Microsoft.Owin.*;       TO: (no successor; the file is deleted or rewritten)
Applies to Group C only. Groups A, B and D are governed by their role, not their imports.
```

**Group D — the eight files with NO legacy directive, which do not all port unchanged.** This is the
group a substitution table declares finished, and three of the eight are not:

| File | Reality |
| --- | --- |
| `Properties/AssemblyInfo.cs` | **Disappears** into MSBuild properties — [04 §5.3](04-dotnet8-migration-strategy.md) |
| `ViewModels/ShoppingCartRemoveViewModel.cs` | **Gains explicit JSON property names** on all five properties [:5-9] — section [8.7](#87-the-json-contract--annotate-one-model-not-the-policy). Without them the cart page breaks silently, and this file carries not one legacy import |
| `Models/AccountViewModels.cs` | **Adapted** to the target Identity model shapes, and reduced by whatever the external-login removal of section [8.3](#83-the-five-views-that-name-legacy-types) makes unreachable |
| `Models/MusicStoreEntities.cs` | **Gains an explicit `DbContextOptions` constructor and an `OnModelCreating`** — section [4.5](#45-two-contexts-two-migration-sets-two-history-tables). Its zero legacy imports are exactly why this is the dangerous file: nothing on the page changes appearance, and the class-name connection convention silently stops meaning anything |
| `Models/Album.cs`, `Models/Cart.cs`, `Models/Genre.cs`, `Models/OrderDetail.cs` | **Need explicit EF Core mapping** where EF 6 relied on convention, expressed in `OnModelCreating` and settled against the extracted schema of section [5.1](#51-domain-data-migration-starts-with-schema-extraction). The `virtual` modifiers on `Album.cs:30-32` and `Cart.cs:17` become inert — they were there for EF 6 proxy lazy loading, which the target does not do (section [4.6](#46-lazy-loading--a-resolution-per-site-not-a-policy)) |
| `ViewModels/ShoppingCartViewModel.cs` | A **true straight port** |

**So of the twenty-seven files: seven are deleted, three change identity, ten take substitution plus
per-site work, six need explicit mapping or annotation, and exactly one ports unchanged.** That is the
plan. The import census is how you find the files, not what you do to them.

---

## 10. Administrator provisioning becomes an operator command

### 10.1 What it is today

`private async void CreateAdminUser()` [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:21] runs at
application startup, called from `ConfigureApp` [:18]. It reads two app settings [:23-24], hard-codes
the role name [:25], hand-builds a context, a user manager and a role manager [:27-29], creates the
role if absent [:32-36], and creates the user if absent [:38-44]. MVC 4 does the equivalent through
SimpleMembership [src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:21-37].

Three properties of that make it the wrong shape rather than merely the wrong place:

- **`async void`.** An unhandled exception in an `async void` method has nowhere to propagate, so a
  provisioning failure at startup is **unobservable** — the application starts, appears healthy, and has
  no administrator ([09 §3.6](09-security-assessment.md) owns the finding).
- **A plaintext credential in source control** [src/MVC5/MvcMusicStore/Web.config:16-17], read on every
  start ([09 §3.5](09-security-assessment.md) owns it; [08 §5.6](08-technical-debt-register.md) rates
  it).
- **Partial idempotence.** See property 3 below.

### 10.2 The target: `tools/provision-admin`, and five required properties

A separate console project, `tools/provision-admin/ProvisionAdmin.csproj` and its `Program.cs`. Five
properties are **required**, not implied — each is a design constraint whose absence would reproduce a
defect the source has.

**1. The secret does not arrive on the command line.** A password passed as an argument is visible in
process listings to any user on the host and is recorded by shell history and by pipeline logs, which
would move the credential from one durable store to three. It is supplied either through an
**environment variable scoped to the single invocation**, or by **interactive prompt with terminal echo
disabled**. **The release process uses the environment-variable channel**, injected by the pipeline from
the platform secret store for that one step and not persisted to any file; the interactive prompt is
the path for an operator running the command by hand outside a pipeline. Both are implemented; the
command refuses to read a password from an argument at all, so the unsafe channel is not merely
discouraged.

**2. Identity hashes the password — the command builds a host and resolves the managers from the
container.** It creates a host with the same Identity configuration as the web application, resolves
`UserManager<ApplicationUser>` and the role manager, and calls their create methods. **Direct SQL is
explicitly not an acceptable substitute**, because it cannot produce a valid Identity password hash:
the hash format, iteration count and salt handling are the hasher's, and a hand-written `INSERT`
produces a row that exists and cannot sign in. Sharing the configuration also means the password is
validated against the **same** policy the web application enforces (section
[6.1](#61-the-policy-table-every-row-labelled)), so a provisioned administrator cannot bypass the
complexity rules.

**3. Idempotence is per operation, not overall.** Three checks, evaluated **independently**: create the
user only if absent, create the role only if absent, add the membership only if absent.

**The evidence that this matters is a difference between the two editions.** MVC 4 checks all three
independently — `WebSecurity.UserExists` [src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:29],
`Roles.RoleExists` [:32] and `Roles.IsUserInRole` [:35] — whereas **MVC 5 adds the membership only
inside the user-creation branch**: `AddToRoleAsync` [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:43]
sits inside `if (user == null)` [:39-44]. So if a prior MVC 5 run created the user and then failed
before the membership was added, **every subsequent run skips it** — the user exists, so the branch is
not entered, and the account never gets the role. The administration surface is then permanently
unreachable and nothing reports why. The target repairs such a run rather than skipping it, which is
the whole point of checking the three independently.

**4. The audit record has a destination, and no secret in it.** Actor, timestamp, target username, role
name and outcome — for each of the three operations separately, so "role already existed, membership
added" is distinguishable from "nothing to do" — written through `ILogger` to the application's
configured sink and retained under the platform's log-retention policy. **The password is never
logged**, never echoed, and never included in an exception message. (The collection mechanism is
[06](06-azure-hosting-recommendations.md)'s.) This is net-new: the repository has no audit logging of
any kind ([09 §3.13](09-security-assessment.md)).

**5. It is not deployed with the web application.** A separate project, not referenced by the web
project and not included in its publish output, run from the release pipeline or an operator session.
**The credential leaves configuration entirely** — no `appsettings` key, no environment variable on the
web application, and nothing for a compromised web process to read.

### 10.3 What this retires

The `async void` method [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:21] and its call [:18]; the
three hand-built objects [:27-29]; the two `ConfigurationManager.AppSettings` reads [:23-24] and the
`using System.Configuration;` [:7] that supports them; and the two committed credential keys
[src/MVC5/MvcMusicStore/Web.config:16-17]. **Failure becomes observable** — a non-zero exit code and a
log record from a command someone ran, rather than silence inside a start-up path.

Provisioning moves out of the application and into an operator action, which is an **approved delta**
with **Security and operations** as its approval owners (section
[11.5](#115-the-full-set-of-approved-deltas)). The user-visible consequence is that a freshly deployed
environment has no administrator until the command is run, which the deployment runbook of
[06](06-azure-hosting-recommendations.md) must include as a step.

---

## 11. The cutover decision

### 11.1 The decision

**A single cutover, not a strangler fig.** The ported application replaces the legacy application in
one switch, during a planned window, with a rollback plan.

This is a **decision**, not a comparison, and it is stated once. [03](03-modernization-roadmap.md)
sequences around it. **A reappearance of this question as an unresolved comparison anywhere in the
deliverable set is a defect**, not a nuance — the incremental path appears below only as a conditional
alternative with named trigger conditions.

### 11.2 Why

MvcMusicStore is **one deployable unit** with six controllers and roughly **2,097 non-blank lines** of
C# in the migration source — the **sizing metric** of [08 §2.1](08-technical-debt-register.md), non-blank
lines excluding assembly metadata, which is the method [08 §4.1](08-technical-debt-register.md) uses
for that figure and the only method under which it should be read. It has:

- **no independently deployable module** — one project, no `ProjectReference`, no areas, and one
  conventional route [src/MVC5/MvcMusicStore/App_Start/RouteConfig.cs:16-20] serving the whole URL
  surface;
- **no route family migratable in isolation.** The layout renders the cart summary and the genre menu
  on **every** page [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:25-26], and the cart identity
  lives in session [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:161-180], so migrating any route
  drags session and identity across the boundary with it;
- **no traffic profile that makes a partial cutover safer** than a whole one — there is no measured
  load, no SLA and no observability to measure one with ([11 §3.8](11-cloud-readiness-assessment.md)).

The strangler path would require a second host, adapters on both sides, shared session across the
boundary and authentication continuity across it — real, permanent-feeling complexity, built and then
discarded, in exchange for a granularity this application cannot use.

### 11.3 What single cutover does not buy

It does not remove the need for the pre-port test suite; it increases it, because there is no
route-by-route soak period in which a regression surfaces on a fraction of traffic. Section
[12](#12-the-test-suite-architecture-and-coverage) is the compensating control, and it is why tests
precede the port rather than accompanying it.

### 11.4 Two accepted costs

Both follow from facts already established, and both are accepted **deliberately here** rather than
discovered on the day.

**Cost one — every signed-in user is signed out at cutover.**

Katana cookie authentication tickets are protected by the old application's key material, and the new
application has a different key ring; there is no shared key material in the repository to bridge them
(no `machineKey` in any edition — [11 §3.2](11-cloud-readiness-assessment.md)). **The tickets are
unreadable by the new runtime.** The adapters' remote-authentication mechanism exists precisely to
avoid this and is not in play under a single cutover.

The response is **operational, not technical**:

- **Expire the legacy authentication cookie at the new application's first request**, so no browser
  retries an undecryptable ticket and then presents an ambiguous state. This is a deliberate deletion
  of the old cookie by name, not a reliance on the framework ignoring it.
- **Notify users ahead of the window.**
- **Accept re-authentication as the cost.** [06](06-azure-hosting-recommendations.md) states it in the
  cutover runbook.

**Cost two — anonymous carts do not survive, even though their rows do.**

An anonymous cart's identity lives **only** in in-process session: a `Guid.NewGuid()` written to
session and never sent to the client as anything else
[src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:161-180], specifically [:172] and [:175]. When the old
process stops, **the browser-to-GUID link is lost**, so the `Cart` rows become **orphaned regardless of
whether they are migrated** — nothing can ever look them up again.

**Signed-in carts are unaffected**, because their key is the login name [:167], which the Identity data
migration preserves as its invariant (sections
[4.4](#44-what-the-cart-is-coupled-to--and-what-it-is-not) and
[5.5](#55-the-identity-transition--a-schema-decision-plus-a-data-migration)).

The response:

- **Schedule cutover in a low-traffic window** and **drain the legacy application** before switching, so
  the number of live anonymous sessions at the moment of the switch is as close to zero as scheduling
  can make it.
- **Migrate the `Cart` rows anyway.** No data is destroyed, and the orphaned rows can then be reported
  and cleaned up on a stated schedule afterwards rather than silently vanishing.
- **Building a session-bridging mechanism is explicitly rejected as disproportionate** — it would mean
  standing up shared session state between two frameworks in order to preserve anonymous shopping
  carts on a store whose checkout requires authentication anyway
  [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:8].

**Both costs, plus the maintenance window and the drain step, belong in the rollback plan — because a
cutover that must be reversed produces the same two effects in the other direction.** Reverting signs
everyone out again, and anonymous carts created on the new application are lost on the way back. The
rollback plan must therefore state: the window, the drain, the legacy redeployment, the unmodified
source database as the data position (section
[5.1](#51-domain-data-migration-starts-with-schema-extraction)), the second forced re-authentication,
and the second anonymous-cart loss. A rollback that is written as if it were free is a rollback nobody
will be willing to execute at 2am.

### 11.5 The full set of approved deltas

Behaviour preservation is the baseline: a visitor can browse genres and albums, view album detail, add
to and remove from a cart, see a cart count and summary, register, sign in and out, place an order with
promo-code handling, view their own orders, and — as an administrator — create, edit and delete albums.
Every one of those outcomes is preserved.

**The table below is everything that deliberately changes.** Its purpose is to let validation
distinguish **a preserved contract from an approved change**, rather than treating every difference as
a regression — and equally to stop an approved change being used to excuse a real one.

| # | Approved delta | Why | Section | Approval owner |
| --- | --- | --- | --- | --- |
| 1 | **`AddToCart` changes from `GET` to a token-protected `POST`.** The path is unchanged; the bookmarkable URL stops working | A state-changing `GET` cannot be protected against cross-site request forgery under any policy | [7.3](#73-problem-two--one-state-changing-action-is-a-get-which-no-policy-can-cover) | **Security** |
| 2 | **Administrator provisioning moves from application startup to an operator command.** A fresh environment has no administrator until the command runs | Removes a committed plaintext credential and makes provisioning failure observable | [10](#10-administrator-provisioning-becomes-an-operator-command) | **Security and operations** |
| 3 | **Automatic destructive schema creation becomes deployment-time migrations plus guarded non-production seeding** | `DropCreateDatabaseIfModelChanges` destroys orders and PII on any model change | [5.3](#53-who-applies-ddl-and-in-what-order), [5.4](#54-seeding--the-guard-fails-closed-on-three-checks) | **Data owner** |
| 4 | **Bundling is removed; the five bundle virtual paths cease to exist** | No successor supports the `{version}` and glob token forms | [8.1](#81-bundling-and-static-assets--no-bundler) | **Engineering** |
| 5 | **Bootstrap 3 markup is migrated to the equivalents of its pinned successor** ([04 §9.2](04-dotnet8-migration-strategy.md) owns the pin) | Bootstrap 3 is out of support; the visual-preservation criterion bounds the change | [8.5](#85-the-bootstrap-upgrade-is-markup-work-not-a-version-bump) | **Product and engineering** |
| 6 | **Glyphicons are removed and not replaced** — two decorative icons | Dropped from Bootstrap after 3; replacing them would add a dependency, an acquisition mechanism and a licence question | [8.5](#85-the-bootstrap-upgrade-is-markup-work-not-a-version-bump) | **Product** |
| 7 | **The legacy browser matrix narrows** — the media-query polyfill and the feature-detection library are dropped | Both exist for Internet Explorer 8-era browsers. **[06](06-azure-hosting-recommendations.md) states the target matrix; [07](07-effort-risks-sequencing.md) carries the compatibility risk** | [8.1](#81-bundling-and-static-assets--no-bundler) | **Product** |
| 8 | **Cutover forces re-authentication** | Katana tickets are unreadable by the new key ring | [11.4](#114-two-accepted-costs) | **Product and operations** |
| 9 | **Anonymous carts do not carry across cutover** | Their identity exists only in in-process session | [11.4](#114-two-accepted-costs) | **Product and operations** |
| 10 | **Authentication, cookie, password and lockout policy is set explicitly, and each divergence from today's effective behaviour is labelled hardening** | A silent framework default is a behaviour change nobody approved | [6.1](#61-the-policy-table-every-row-labelled) | **Security** |
| 11 | **The scaffolded-but-disabled external-login surface is removed**, including its actions, views and the set-password branch | Nothing was reachable; it is deployed attack surface | [8.3](#83-the-five-views-that-name-legacy-types) | **Product and security** |
| 12 | **The error response changes shape**: a generic page plus a trace identifier, with the status code preserved and nothing disclosed | The source's error policy is a type that does not exist, and custom errors were never enabled — the behaviour is chosen, not preserved | [8.3](#83-the-five-views-that-name-legacy-types) | **Engineering** |
| 13 | **The sign-out control stops being a `javascript:` URL** and becomes a submit control | An inline-script URL is incompatible with the security headers the port adds | [6.2](#62-the-sign-out-surface) | **Engineering** |
| 14 | **HTTPS is enforced and security response headers are added** | The application is served over plain HTTP with no such header today | [2.4](#24-what-programcs-contains-in-order), [11 §3.6](11-cloud-readiness-assessment.md) | **Security** |

Everything **not** in that table is a preserved contract, and a difference in it is a regression.

### 11.6 The incremental path — a conditional alternative

The incremental path is **available and not selected**. It becomes the right answer only if one of the
following holds — stated as **operational conditions**, not as durations, because a duration is
[07](07-effort-risks-sequencing.md)'s to state:

- **The application must remain continuously available throughout the port, with no maintenance window
  available at any point.** Single cutover requires a window; if none can be obtained, the decision
  changes.
- **The Identity migration must be proven against live production traffic before it is committed.** The
  incremental path lets one host remain the authentication authority while the other is exercised.
- **A risk policy requires route-by-route isolation with independent rollback per route.**

Two facts make it genuinely available rather than theoretical: **MVC 5 targets the framework version
recorded at [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:16]**, which
clears the System.Web adapters' framework floor, and the reverse proxy supports the target framework.
[04 §7.3](04-dotnet8-migration-strategy.md) owns the conditional package pins for both, and this
document names no version.

**If it is selected, sign-in continuity is provided by the adapters' remote authentication** —
delegating authentication to the .NET Framework application so that **one host remains the single
authority** for the duration of the port. That is chosen over cookie-format interoperability, which
would require matching cookie names, authentication schemes, application names and key rings across two
frameworks and would fail silently and intermittently when any one of the four drifted. Remote
authentication has one authority and one failure mode.

Selecting it also **removes cost one and cost two of section [11.4](#114-two-accepted-costs)** and adds
its own: two hosts to operate, two deployment pipelines, an adapter surface on both sides, and a
proxy hop on every request for the duration. That trade is the reason it is the alternative rather than
the plan.

---

## 12. The test suite: architecture and coverage

### 12.1 Why tests precede the port

**The repository contains no test of any kind.** Two independent searches confirm it:

```bash
git ls-files | grep -ic test                                                          # -> 0
git grep -l -E "TestClass|\[Fact\]|\[Theory\]|xunit|NUnit|MSTest" -- '*.cs' '*.csproj' '*.config'   # -> 0
```

([08 §7.3](08-technical-debt-register.md) owns this as debt.) So **nothing that exists today would
detect a behaviour change** — and section [4.6](#46-lazy-loading--a-resolution-per-site-not-a-policy),
section [8.7](#87-the-json-contract--annotate-one-model-not-the-policy) and the eight silent-runtime
blockers of [12 §4](12-migration-blockers.md) are precisely the kind that produce a 200 response and a
wrong outcome.

Combined with the single-cutover decision (section [11.3](#113-what-single-cutover-does-not-buy)), which
gives no partial-traffic soak period, **the suite is the only control that stands between this port and
a silent regression. It is written before the port, against the legacy application, so that it
characterizes the baseline rather than the result.** [03](03-modernization-roadmap.md) places it;
[07](07-effort-risks-sequencing.md) carries the absent baseline as a risk.

### 12.2 The architecture: HTTP-level black box

**The suite asserts over HTTP** — status codes, redirect targets, JSON payloads and selected rendered
content — **not on internal types.** That is not a preference; it is the property that lets **one suite
run against two different runtimes**. A test that references `System.Web.Mvc.ActionResult` can only run
against the source; a test that issues `GET /Store/Details/1` and asserts on the response runs against
both, and comparing the two runs is the entire value.

Project shape (test-framework pins are [04 §7.2](04-dotnet8-migration-strategy.md)'s; this document
names none):

- `src/MvcMusicStore.Tests/MvcMusicStore.Tests.csproj`
- `src/MvcMusicStore.Tests/LegacyBaselineFixture.cs` — drives the **MVC 5** application over HTTP and
  captures normalized responses
- `src/MvcMusicStore.Tests/CoreApplicationFixture.cs` — hosts the ported application in-process against
  a disposable database
- `src/MvcMusicStore.Tests/Contracts/**` — per-surface assertions, run against either fixture

### 12.3 Three problems that must be handled explicitly

**Problem one — the legacy baseline needs BOTH databases, not one.** MVC 5 has a catalog store **and** a
separate Identity store, and both are file-attached from `App_Data`
[src/MVC5/MvcMusicStore/Web.config:12-13]. The directory holds exactly four files — two `.mdf`/`.ldf`
pairs (`git ls-files 'src/MVC5/MvcMusicStore/App_Data/*'`). So a deterministic reset **restores both
pairs** and reattaches them before each run. A fixture that resets only the catalog pair produces a
credential store that accumulates state across runs, and the sign-in assertions then pass or fail
depending on run order.

Two constraints on that reset: it operates on **copies outside the checkout**, because attaching the
tracked files causes the engine to write to them and dirty the working tree; and the Identity database
carries a fixed catalog name in its connection string [:12], so **concurrent fixtures must rename it**
or they collide inside one shared engine instance.

**Problem two — the Core fixture needs a real SQL Server.** The in-process integration host supplies no
database, and **two** target features require a SQL Server-compatible engine rather than an in-memory
substitute: **migrations** (section [5.3](#53-who-applies-ddl-and-in-what-order)) and the **SQL-backed
session cache** (section [6.3](#63-session-and-cart-identity)). An in-memory provider would apply no
migrations and would leave the cart identity untested, which is the mechanism most of the coverage below
depends on.

So the fixture **provisions a disposable SQL Server instance** — a container locally, a throwaway
managed database in CI — and then:

1. creates the session cache table, then applies **both** migration sets in the order of section
   [5.3](#53-who-applies-ddl-and-in-what-order), then creates the data-protection key table;
2. loads a **fixture dataset**: the same catalog rows the seed produces, plus a small set of **migrated**
   users, roles, carts and orders — migrated, so that the Identity data migration of section
   [5.5](#55-the-identity-transition--a-schema-decision-plus-a-data-migration) is under test rather than
   bypassed by freshly created accounts;
3. **asserts row counts and key invariants after loading** — the load itself is a fixture that can fail
   silently, and a suite built on an unverified fixture reports the fixture's bugs as the application's;
4. tears the instance down afterwards.

**Isolation is per test class, against a freshly provisioned database.** Per-test provisioning is too
slow at this fixture size and shared-database isolation leaks through the cart and order tables, which
several tests write.

**Problem three — raw response bodies cannot be compared byte-for-byte.** Anti-forgery tokens, session
identifiers, authentication cookies and timestamps vary per request, and the port **deliberately**
changes the `AddToCart` verb, the Bootstrap markup, the error view and the Identity views. A byte
comparison would fail on every page for reasons that are all either noise or approved.

**Assertions are therefore semantic**: status code, redirect target, JSON property names and values,
and the presence and value of **named** elements located by `id` — of which the cart page conveniently
has several already: `#cart-summary` [src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:47],
`#cart-total` [:87], `#update-message` [:44] and the per-row `#row-{id}` and `#item-count-{id}` [:62],
[:70]. Volatile values are **normalized out** before comparison, and **the approved deltas of section
[11.5](#115-the-full-set-of-approved-deltas) are enumerated in the suite as expected differences**, so
that an approved change reads as an approved change rather than as a failure — and, equally, so that a
difference *not* on that list fails.

**One baseline caveat carries into every comparison**: the source runtime is in 4.5 quirks mode
[src/MVC5/MvcMusicStore/Web.config:34] while the target has no quirks mode at all (section
[3.1](#31-configuration-webconfig-becomes-appsettingsjson-read-through-iconfiguration)). The baseline
capture records this, and any difference traceable to it is classified before it is filed.

### 12.4 Required coverage

Every row is required. Each names what it protects, so that a reader can see why it is not optional.

| # | Surface | Assertions | Protects |
| --- | --- | --- | --- |
| 1 | **Catalog browse** — `/Store`, `/Store/Browse?genre=…` | 200; the genre list; a browse page listing that genre's albums | The `Include("Albums")` string form [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:30] still resolving |
| 2 | **Album detail** — `/Store/Details/{id}` | 200; **the genre name and the artist name are present and correct** | Lazy-loading site a1 (section [4.6](#46-lazy-loading--a-resolution-per-site-not-a-policy)). A missing `Include` renders empty here and this assertion is the only thing that notices |
| 3 | **Admin album detail** — `/StoreManager/Details/{id}` | 200; artist and genre names present | Lazy-loading site a2 |
| 4 | **Cart add under its new verb** — `POST /ShoppingCart/AddToCart/{id}` | Redirect to the cart; the item is in the cart; **and `GET` on the same path does not mutate** | Approved delta 1 (section [11.5](#115-the-full-set-of-approved-deltas)). The negative half is the important half |
| 5 | **Cart remove — valid token** | 200; **JSON property names are exactly `Message`, `CartTotal`, `CartCount`, `ItemCount`, `DeleteId`**, with correct values; the row is gone | The camelCase flip (section [8.7](#87-the-json-contract--annotate-one-model-not-the-policy)) and lazy-loading site a3 |
| 6 | **Cart remove — missing token** | Rejected; **the cart is unchanged** | The header contract (section [7.4](#74-problem-three--cart-removal-posts-without-a-form)) |
| 7 | **Cart remove — invalid token** | Rejected; **the cart is unchanged** | As above. Status-only assertions would pass a rejection that still mutated |
| 8 | **Cart count and summary from the shared layout** | On an arbitrary page, the count element is present and correct; the title text lists the album names | Lazy-loading site a4 — the layout-level one, whose failure is every page, and the view component's own view context (section [8.2](#82-child-actions--three-view-components)) |
| 9 | **Genre menu from the shared layout** | On an arbitrary page, the menu renders with the expected number of entries; **the generated SQL is inspected once** | Lazy-loading category (c): the nested aggregate throws in the target rather than client-evaluating |
| 10 | **Checkout — valid promo code** | Redirect to completion; **an order exists whose `Total` equals the sum of its order-detail rows** | Lazy-loading site a6, the financial one, and the input model |
| 11 | **Checkout — invalid promo code** | The form is redisplayed; **no order is created and the cart is unchanged** | The promo-code comparison [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:33-34] |
| 12 | **Checkout — missing promo code** | Same as row 11 | The ten-versus-nine property trap (section [8.8](#88-the-checkout-input-model--ten-properties-not-nine)). A nine-property model makes rows 11 and 12 pass and row 10 fail — which is why all three are required |
| 13 | **Checkout — over-posting attempt** | A post including `Total`, `Username`, `OrderId` or `OrderDate` does **not** set them from the request | The include list's successor (section [8.8](#88-the-checkout-input-model--ten-properties-not-nine)); [09 §6.4](09-security-assessment.md) owns the finding |
| 14 | **Order ownership** — `/Checkout/Complete/{id}` | The owner sees the confirmation; **another authenticated user does not** | [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:71-73], and the username invariant of section [5.5](#55-the-identity-transition--a-schema-decision-plus-a-data-migration) |
| 15 | **Administration authorization** | **A permitted identity** (administrator) reaches `/StoreManager` with 200; **a denied identity** (authenticated non-administrator) does not; anonymous is redirected to login | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:12], and the role membership the Identity migration must preserve |
| 16 | **Administration writes** | Create, edit and delete each succeed with a valid token and are **rejected without one**, leaving the album unchanged | The three previously unprotected POSTs (section [7.2](#72-problem-one--coverage-today-is-partial)) |
| 17 | **Not-found behaviour** | `/StoreManager/Details/{missing}` returns **404**, not 200 and not 500 | The three `NotFound()` conversions, and the error policy's status-code preservation (section [8.3](#83-the-five-views-that-name-legacy-types)) |
| 18 | **Identity migration and sign-in of a pre-existing account** | A **migrated** account signs in with its original password; its stored hash is observed rewritten in the target format; its order history is visible | Section [5.5](#55-the-identity-transition--a-schema-decision-plus-a-data-migration). **This is the acceptance test for the hash-format assumption** |
| 19 | **Sign-in cart migration, both failure paths** | Happy path: the anonymous cart becomes the user's. Failure path: cart migration failing leaves the user **signed in** with a notice; an Identity-side failure leaves the cart **unreassigned** | The invariant of section [4.3](#43-the-cross-store-consistency-model) |
| 20 | **Register, sign in, sign out** | Registration creates an account under the target's password policy; sign-out is a **token-validated POST** and the ticket is expired afterwards | Section [6.2](#62-the-sign-out-surface) |
| 21 | **Login partial, both branches** | Signed-out renders register and log-in links; signed-in renders the greeting with the user's name and the sign-out form | The null-handled identity check (section [8.3](#83-the-five-views-that-name-legacy-types)), whose failure mode is rendering the wrong branch silently |
| 22 | **Removed external-login routes** | Every removed route returns **404**, and sign-in, register, sign-out and manage are unaffected | The removal decision (section [8.3](#83-the-five-views-that-name-legacy-types)) |
| 23 | **Static assets** | Every asset referenced by a rendered page returns 200 — **case-sensitively** | The `site.css`-versus-`Site.css` mismatch (section [8.1](#81-bundling-and-static-assets--no-bundler)); a case-insensitive check passes on the wrong filesystem |
| 24 | **The non-production seeding guard** | A seeding attempt against a **production-shaped configuration** fails **and writes nothing** — row counts identical before and after | Section [5.4](#54-seeding--the-guard-fails-closed-on-three-checks) |
| 25 | **Cookie continuity across replicas and across a restart** | A cookie issued by one instance is accepted by another; a cookie issued before a restart is accepted after it | The persisted key ring (sections [2.4](#24-what-programcs-contains-in-order), [6.1](#61-the-policy-table-every-row-labelled)); [11 §3.2](11-cloud-readiness-assessment.md) owns the requirement |
| 26 | **Session continuity across replicas** | An anonymous cart built against one instance is visible from another | The distributed cache (section [6.3](#63-session-and-cart-identity)) |
| 27 | **Deployment health check** | The health endpoint returns healthy with the database reachable, and unhealthy when it is not | Net-new capability; there is no health endpoint today ([11 §3.8](11-cloud-readiness-assessment.md)) |

Rows 1 to 22 run against **both** fixtures — the legacy baseline and the ported application — except
where an approved delta makes the legacy shape different, in which case the baseline records the old
shape and the suite records the delta. Rows 23 to 27 are target-only, because the source has no
equivalent behaviour to characterize.

### 12.5 Rendered-appearance comparison is separate, and manual

**The visual-preservation criterion of section
[8.5](#85-the-bootstrap-upgrade-is-markup-work-not-a-version-bump) cannot be met by the suite above**,
and saying otherwise would be the most consequential overclaim in this document. An HTTP-level suite
asserts on response **content**; the Bootstrap major-version move is a change in how that content
**renders**. A test
asserting that a `<div>` carries a given class does not know whether the navigation bar looks right.

**Automating it is rejected**, deliberately: it would mean pinning a browser-automation stack and
screenshot tooling, defining viewports, baseline images and pixel tolerances, and storing binary image
artifacts in a repository that already carries 43 MB of committed database binaries
([08 §6.2](08-technical-debt-register.md)) — infrastructure this application needs for **nothing else**,
bought for a **one-time** layout migration.

**The decision is a manual review against a captured baseline:**

- **Screenshots of every distinct page**, taken from the MVC 5 application **before** the port, at the
  **two viewports the layout actually distinguishes** — the navbar collapses at one breakpoint
  [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:15], [:22], and the viewport meta tag [:5]
  confirms the responsive intent — so one wide and one narrow capture per page.
- **A reviewer checklist** covering the **navigation bar** (brand, links, collapse behaviour, the cart
  badge, the fixed-top position), the **catalog grid**, **album detail**, the **cart table**, the
  **checkout form** (label and control alignment, validation-message placement) and the
  **administration list**.
- **A signed-off approval artifact** recording **who reviewed it**, against which baseline, and **which
  of the approved deltas of section [11.5](#115-the-full-set-of-approved-deltas) were accepted** —
  specifically deltas 5, 6 and 13, which are the visible ones.

**The automated suite does not claim to cover this.** [07](07-effort-risks-sequencing.md) carries the
review as a task.

---

## 13. Resolution register — all 22 blockers of deliverable 12

This deliverable's acceptance criterion is that **every no-successor construct and every
differing-default item in deliverable 12 has a named resolution rather than a description.** This
section is the audit trail — the criterion is recorded in the requirement map at
[README](README.md). It
walks [12](12-migration-blockers.md)'s two groups in order. A row's *Resolution* column names the
decision; the *Where* column points at the section that specifies it.

### 13.1 Group one — no successor, or the API itself is gone (compile-time)

| ID | Blocker | Resolution | Where |
| --- | --- | --- | --- |
| F-12-01 | SQL Server Compact 4.0 as MVC 3's catalog provider | **Out of this port's scope by triage, and recorded as such rather than left blank.** MVC 3 is not the migration source; it is retained read-only. No provider is selected for it and no port of its data layer is specified — [03](03-modernization-roadmap.md) owns the triage, [10](10-build-and-deployment-requirements.md) the topology | — |
| F-12-02 | `System.Web.Optimization` bundling, `{version}` and glob tokens | **No bundler.** Assets move to `wwwroot`, served by static-file middleware, cache-busted by the version-appending tag helper; no JavaScript toolchain is introduced; all 11 helper call sites are rewritten; the `site.css` casing is corrected in the move | [8.1](#81-bundling-and-static-assets--no-bundler) |
| F-12-03 | The Katana `IAppBuilder` abstraction and the OWIN startup attribute | **The abstraction is deleted; the responsibilities are re-registered explicitly in `Program.cs`.** Cookie authentication becomes a framework handler with **every policy value stated** rather than inherited; the external sign-in cookie is removed with the surface | [2.3](#23-owin-startup-the-abstraction-goes-the-responsibilities-do-not), [2.4](#24-what-programcs-contains-in-order), [6.1](#61-the-policy-table-every-row-labelled) |
| F-12-04 | `System.Web.HttpApplication` and the `Global.asax` markup | **Disposition per responsibility, not per file.** The markup file and the base type end with no counterpart; the five registrations get five separate resolutions — deleted, replaced, moved, deleted outright, replaced | [2.1](#21-disposition-of-every-current-startup-and-configuration-file--twelve-files-four-fates), [2.2](#22-the-five-application_start-registrations-do-not-share-one-fate) |
| F-12-05 | `HandleErrorAttribute` — the entire error-handling policy | **Exception-handling middleware, with the policy written rather than migrated** — because custom errors were never enabled, so there is no live behaviour to preserve. The vacated global-filter slot is taken by the anti-forgery filter | [8.3](#83-the-five-views-that-name-legacy-types), [7.1](#71-the-policy) |
| F-12-06 | `System.Web.Mvc.HandleErrorInfo` as a view model | **Define `ErrorViewModel.cs`**, and specify the error route, the status-code behaviour for both unhandled exceptions and status-code responses, what is logged, and what may be disclosed. The view's user-facing text is preserved | [8.3](#83-the-five-views-that-name-legacy-types) |
| F-12-07 | The synchronous `TryUpdateModel` and the class-level `[Bind]` | **`Binding/CheckoutInputModel.cs` with TEN properties** — the nine bound plus `PromoCode`, which is **not** added to the `Order` entity. `TryUpdateModel` becomes model binding on an explicit input model. Tested with valid, invalid and missing promo codes | [8.8](#88-the-checkout-input-model--ten-properties-not-nine) |
| F-12-08 | Three `HttpNotFound()` calls | **`NotFound()`** at [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:35], [:76] and [:108]. The **fourth** `Find` in the same controller, `DeleteConfirmed` [:119-120], has no null check today, compiles and throws identically in the target, and is therefore **not** changed by this port — [08 §5.4](08-technical-debt-register.md) owns it as debt. Coverage row 17 asserts the 404 | [12.4](#124-required-coverage) |
| F-12-09 | `ChallengeResult` and its `ExecuteResult` override | **DECIDED: remove the disabled external-login surface entirely and delete the type**, along with the `IAuthenticationManager` property, the `XsrfKey` constant, `RemoveAccountList`, the five external-login actions and their five views, the external sign-in cookie and the set-password branch. Nothing reachable is lost. Route tests assert 404 on the removed routes | [8.3](#83-the-five-views-that-name-legacy-types), [8.2](#82-child-actions--three-view-components) |
| F-12-10 | `BlockViewHandler` and the Razor host section group | **Split: the handler mapping ends with no replacement** — the target does not serve content-root `.cshtml` — **and the namespace registration becomes `Views/_ViewImports.cshtml`**, with the three surviving namespaces remapped and the three dead ones dropped, named individually | [8.6](#86-view-serving-and-_viewimports) |
| F-12-11 | The `.axd` ignore route | **Dropped, not replaced** — no `.axd` handler exists in the target, so there is nothing to exclude. The single conventional route it accompanies becomes `{controller=Home}/{action=Index}/{id?}` | [2.2](#22-the-five-application_start-registrations-do-not-share-one-fate) |
| F-12-12 | Assembly metadata in a source file | **Absorbed into MSBuild properties.** [04 §5.3](04-dotnet8-migration-strategy.md) owns the mechanism; this document records the file's fate in the disposition table | [2.1](#21-disposition-of-every-current-startup-and-configuration-file--twelve-files-four-fates) |
| F-12-13 | Windows-authentication connection strings and file-attached databases | **The configuration-reading transition is this document's and is specified**: `Web.config` ceases to be a configuration source, `ConfigurationManager.AppSettings` has no successor and its two reads are removed outright, and both contexts take their connection from `IConfiguration`. **The connection content, the identity model and the secret-delivery mechanism are [06](06-azure-hosting-recommendations.md)'s** | [3.1](#31-configuration-webconfig-becomes-appsettingsjson-read-through-iconfiguration), [4.5](#45-two-contexts-two-migration-sets-two-history-tables) |
| F-12-14 | MVC 4's committed build configuration | **Out of this port's scope by triage.** MVC 4 is not the migration source and is not repaired. [10](10-build-and-deployment-requirements.md) owns the diagnosis, [07](07-effort-risks-sequencing.md) the baseline-availability consequence | — |

### 13.2 Group two — the successor exists and its default differs (silent runtime)

| ID | Blocker | Resolution | Where |
| --- | --- | --- | --- |
| F-12-15 | Lazy loading — **nine sites, three categories** | **Explicit eager loading or projection at each site; no proxy package.** Category (a), six sites: a1 and a2 replace `Find` with a query carrying typed `Include`s; a3 and a4 **project** instead of loading; a5 takes an `Include`; **a6 computes the order total from the album the loop already loads**. Category (b), one site: **verify the translation and add no `Include`.** Category (c), two sites: verify translation, rewrite as a grouped projection if it fails, inspect the generated SQL once. The string `Include("Albums")` stays valid | [4.6](#46-lazy-loading--a-resolution-per-site-not-a-policy) |
| F-12-16 | JSON property naming flips to camelCase | **DECIDED: annotate that ONE response model with explicit JSON property names.** The application-wide serializer policy is **not** changed; the JavaScript is **not** changed. Newtonsoft.Json's removal is an unused-dependency removal, not a serializer replacement. Coverage row 5 asserts on property names, not on the status code | [8.7](#87-the-json-contract--annotate-one-model-not-the-policy) |
| F-12-17 | Filesystem path casing | **Every asset reference is emitted against the actual filename during the move to `wwwroot`, and coverage row 23 asserts asset availability case-sensitively.** The repository-wide audit that gates the hosting recommendation is [06](06-azure-hosting-recommendations.md)'s; [11 §3.7](11-cloud-readiness-assessment.md) is the finding of record | [8.1](#81-bundling-and-static-assets--no-bundler), [12.4](#124-required-coverage) |
| F-12-18 | The 4.8-versus-4.5 quirks-mode mismatch | **The concept ends — there is no quirks mode in the target — so the resolution is procedural and belongs to the baseline: the capture records that the source ran in 4.5 quirks mode, and any difference traceable to it is classified before it is filed as a regression.** Both misreadings are named so neither is available by default | [3.1](#31-configuration-webconfig-becomes-appsettingsjson-read-through-iconfiguration), [12.3](#123-three-problems-that-must-be-handled-explicitly) |
| F-12-19 | Connection-string resolution by convention, two conventions | **DECIDED: both contexts get an explicit `DbContextOptions` constructor and an explicit scoped registration; the two contexts REMAIN SEPARATE on ONE database, with separate migrations folders and distinct history tables.** `MusicStoreEntities` also gains an `OnModelCreating`, because the class-name convention was not its only implicit behaviour. The one-database reason is **convention, not a foreign key**, and the trade is tabulated | [4.5](#45-two-contexts-two-migration-sets-two-history-tables), [4.3](#43-the-cross-store-consistency-model) |
| F-12-20 | `Dispose` overrides on objects a container will own | **Delete both overrides in the migration source** — [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:125-128] and [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:324-331]. There is no replacement construct. The account controller's goes with the wholesale rewrite. The MVC 4 and MVC 3 overrides are out of scope by triage | [4.7](#47-the-dispose-overrides-must-be-removed), [9.2](#92-the-four-groups) |
| F-12-21 | The Identity schema is not knowable from the repository | **An authoritative `sys.columns` extraction is a GATE on the Identity migration**, and the migration itself is specified in full: fresh tables, usernames as the invariant, optional id preservation labelled optional, normalization collision detection that **stops** rather than dropping an account, a defined origin for every new column, hash verification with a stated fallback, and reconciliation with the source retained until it passes. **The evidence qualification is [12](12-migration-blockers.md)'s and is cited, not re-derived** | [5.5](#55-the-identity-transition--a-schema-decision-plus-a-data-migration) |
| F-12-22 | No usable schema baseline exists | **Extraction from the live catalog database, then a generated-schema diff that must pass before any data is loaded.** Neither `MvcMusicStore-Create.sql` copy is used. Load order, per-table row reconciliation and per-order financial reconciliation are specified, as is the rollback position | [5.1](#51-domain-data-migration-starts-with-schema-extraction) |

### 13.3 The favourable findings, and where they are used

[12 §6](12-migration-blockers.md)'s four portability findings are consumed rather than restated:
`HttpContext.Current` appearing nowhere is why the service extraction of section
[4.8](#48-the-service-layer-and-the-two-patterns-deliberately-not-adopted) is a signature change rather
than an excavation; the dead `Controller` overload being provably unreferenced is why it is deleted
there with zero call-site impact; the absence of raw SQL anywhere is why section
[4.6](#46-lazy-loading--a-resolution-per-site-not-a-policy) needs no SQL-dialect review; and the async
posture — no `.Result` and no `.Wait()` in any edition — is why the `TryUpdateModelAsync` conversion of
section [8.8](#88-the-checkout-input-model--ten-properties-not-nine) has no sync-over-async debt behind
it.

### 13.4 The three transformations no tool infers

Called out separately because each is a **required step** rather than an observation, and none of them
is produced by any automated upgrade tool:

1. **The ten-versus-nine property fact of the checkout input model.** A tool reading the `[Bind]`
   attribute produces nine. Nine compiles, binds, persists, and silently disables the promo code —
   section [8.8](#88-the-checkout-input-model--ten-properties-not-nine).
2. **The per-site lazy-loading decisions, including the one site that must *not* get an `Include`.** No
   tool can distinguish a client-side dereference from a navigation read inside a translated expression
   — section [4.6](#46-lazy-loading--a-resolution-per-site-not-a-policy).
3. **The PascalCase JSON contract between one endpoint and one script.** No tool reads a JavaScript
   property access and infers a serializer constraint on a C# model — section
   [8.7](#87-the-json-contract--annotate-one-model-not-the-policy).

A fourth is worth the same status even though it is not a code transformation: **the authoritative
schema extraction gate** (sections [5.1](#51-domain-data-migration-starts-with-schema-extraction) and
[5.5](#55-the-identity-transition--a-schema-decision-plus-a-data-migration)). Every tool will happily
generate an initial migration; none will tell you it does not match the database your data is in.

---

## 14. Roll-up

### 14.1 The approach in ten statements

1. **One composition root.** Twelve startup and configuration files collapse into `Program.cs` plus
   Razor and MSBuild conventions, with four different fates and no file's fate inferred from its
   imports.
2. **Nothing is applied at startup that changes a schema.** Migrations, seeding and administrator
   provisioning all leave the web application.
3. **Two contexts, one database, separate migration sets and history tables.** The stores are coupled
   by convention, not by a foreign key, and the consistency model that keeps them safe is an explicit
   five-step ordering with a stated invariant.
4. **Every silent-failure site is named individually.** Nine lazy-loading sites in three categories,
   one JSON contract, two `Dispose` overrides, one connection convention — resolved per site, not by
   policy.
5. **Data migration begins with extraction and a schema diff that must pass.** An initial migration
   creates tables; it does not move rows and it cannot be trusted to match.
6. **The Identity transition preserves usernames as an invariant**, treats id preservation as an
   explicit convenience, detects normalization collisions before writing, and verifies rather than
   assumes the password-hash format.
7. **Policy is stated, not inherited.** Every authentication, cookie, password and lockout value is set
   explicitly and labelled preserved or hardening — including the two places this document deliberately
   declines to harden.
8. **Anti-forgery is global by default**, one state-changing `GET` becomes a `POST`, and the one
   form-less AJAX post gets a named header contract with three token tests.
9. **A single cutover, with two accepted costs and a rollback plan that states them in both
   directions.**
10. **Tests precede the port**, HTTP-level and semantic, against a two-database legacy baseline and a
    disposable SQL Server for the target — with rendered-appearance review scoped separately and
    manually.

### 14.2 What this document creates: nothing

No repository file was created, modified or deleted in producing it, other than this file. Every target
artifact named above is a specification for a separately approved implementation phase. The acceptance
check is `git status --porcelain` showing only new files under `docs/modernization/`.

### 14.3 Cross-reference index

| Needed | Go to |
| --- | --- |
| Target framework, SDK band, project format, `PackageReference`, tooling manifests | [04](04-dotnet8-migration-strategy.md) §2, §3, §5, §6 |
| Every package pin — runtime, client-side, tools, tests, conditional incremental | [04](04-dotnet8-migration-strategy.md) §7, §8, §9 |
| The future application map | [04](04-dotnet8-migration-strategy.md) §12 |
| Hosting target, deployment model, cutover runbook, browser matrix | [06](06-azure-hosting-recommendations.md) |
| Telemetry collection, data-protection key-ring location, DDL principal, cache-table provisioning | [06](06-azure-hosting-recommendations.md) |
| Per-edition build outcomes, database topology, view-compilation finding | [10](10-build-and-deployment-requirements.md) |
| Effort model, risk register, the manual visual review as a task | [07](07-effort-risks-sequencing.md) |
| Workstream decomposition, gate placement, edition triage | [03](03-modernization-roadmap.md) |
| Statefulness, data-protection requirement, transport, path casing | [11](11-cloud-readiness-assessment.md) §3 |
| The 22 blockers and their evidence | [12](12-migration-blockers.md) |
| Counting methods, debt severities and owners | [08](08-technical-debt-register.md) §2, §5-§10 |
| Security findings, policy inheritance, anti-forgery coverage, the credential | [09](09-security-assessment.md) §3, §6 |
| Architecture as-is, the two cart unit-of-work models, capability coverage | [01](01-architecture-overview.md) |
| Dependency inventory and the 63 pins | [02](02-dependency-inventory.md) |

---

## Appendix A — Reproducibility

Every repository-wide claim in this document was produced by one of the commands below, run against
this checkout. They are collected here so each is re-runnable rather than merely cited.

```bash
# --- Section 2.2  There is no Areas folder, so RegisterAllAreas() is dead ------
git ls-files | grep -c 'Areas/'                                   # -> 0

# --- Section 3.2  The XDT transforms: 15 occurrences, 3 of them live ----------
# A plain grep counts commented template text too, which is 12 of the 15.
# Strip XML comments first, then count what remains, per file.
python - <<'PY'
import subprocess, re
files = subprocess.run(['git','ls-files','*Web.Debug.config','*Web.Release.config'],
                       capture_output=True, text=True).stdout.split()
tot = live = 0
for f in files:
    s = open(f, encoding='utf-8-sig').read()
    stripped = re.sub(r'<!--.*?-->', '', s, flags=re.S)
    n, m = len(re.findall(r'xdt:Transform', s)), len(re.findall(r'xdt:Transform', stripped))
    tot += n; live += m
    print('%-58s total=%d live=%d' % (f, n, m))
print('TOTAL=%d LIVE=%d' % (tot, live))
PY
# -> 6 files; TOTAL=15 LIVE=3
#    every Web.Debug.config      total=2 live=0
#    every Web.Release.config    total=3 live=1   (the compilation transform, line 18)

# --- Section 3.2 / 8.3  The one live transform is line 18 in all three --------
for f in src/MVC5/MvcMusicStore/Web.Release.config \
         src/MVC4/MvcMusicStore/Web.Release.config \
         src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Web.Release.config; do
  printf '%s:18: %s\n' "$f" "$(sed -n '18p' "$f")"
done
# -> all three:  <compilation xdt:Transform="RemoveAttributes(debug)" />

# --- Section 8.3  No live <customErrors> element in any edition ---------------
python - <<'PY'
import subprocess, re
files = [f for f in subprocess.run(['git','ls-files','*.config'],
         capture_output=True, text=True).stdout.split() if '/packages/' not in f]
tot = live = 0
for f in files:
    s = open(f, encoding='utf-8-sig').read()
    stripped = re.sub(r'<!--.*?-->', '', s, flags=re.S)
    tot += len(re.findall(r'<customErrors', s))
    live += len(re.findall(r'<customErrors', stripped))
print('customErrors total=%d live=%d' % (tot, live))
PY
# -> total=12 live=0

# --- Section 7.2  The POST census, and the trap in counting it ----------------
git grep -c '\[HttpPost\]' -- 'src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs'
# -> 2      WRONG: :116 is [HttpPost, ActionName("Delete")] and does not match
python - <<'PY'
import subprocess, re
files = subprocess.run(['git','ls-files','src/MVC5/MvcMusicStore/Controllers/*.cs'],
                       capture_output=True, text=True).stdout.split()
for f in files:
    # utf-8-sig: all 27 source files carry a BOM, which breaks a naive ^\s* match
    lines = open(f, 'rb').read().decode('utf-8-sig').splitlines()
    at = [i+1 for i, l in enumerate(lines) if re.match(r'\s*\[HttpPost', l)]
    print('%-28s posts=%d at %s' % (f.split('/')[-1], len(at), at))
PY
# -> AccountController.cs      posts=8 at [53, 86, 112, 146, 197, 235, 262, 300]
#    CheckoutController.cs     posts=1 at [25]
#    ShoppingCartController.cs posts=1 at [54]
#    StoreManagerController.cs posts=3 at [53, 86, 116]     <- three, not two
#    HomeController.cs / StoreController.cs  posts=0
git grep -n 'ValidateAntiForgeryToken' -- 'src/MVC5/MvcMusicStore/Controllers/*.cs' | wc -l
# -> 8, all in AccountController.cs  =>  13 POSTs, 8 validated, 5 unprotected

# --- Sections 7.1, 8.1, 8.2, 8.4  View-level helper censuses ------------------
git grep -o -E '@Html\.AntiForgeryToken\(\)' -- 'src/MVC5/*.cshtml' | wc -l   # -> 10
git grep -o '@Scripts\.Render'  -- 'src/MVC5/*.cshtml' | wc -l                # -> 10
git grep -o '@Styles\.Render'   -- 'src/MVC5/*.cshtml' | wc -l                # -> 1
git grep -o '@Html\.Partial'    -- 'src/MVC5/*.cshtml' | wc -l                # -> 5
git grep -o '@Url\.Content'     -- 'src/MVC5/*.cshtml' | wc -l                # -> 4
git grep -n -o '@Html\.Action(' -- 'src/MVC5/*.cshtml'
# -> Views/Account/Manage.cshtml:22, Views/Shared/_Layout.cshtml:25 and :26
git grep -c '@Ajax\.' -- 'src/MVC5/*.cshtml' | wc -l                          # -> 0
git ls-files 'src/MVC5/*.cshtml' | wc -l                                      # -> 29

# --- Section 8.5  Glyphicons: exactly two views, neither of them the layout ---
git grep -n 'glyphicon' -- 'src/MVC5/*.cshtml'
# -> Views/ShoppingCart/CartSummary.cshtml:5
#    Views/Store/Details.cshtml:27

# --- Section 8.1  Path casing, and the 27 asset files ------------------------
git ls-files 'src/MVC5/MvcMusicStore/Content/*'
# -> Content/Site.css   Content/bootstrap.css   Content/bootstrap.min.css
#    the style bundle registers "~/Content/site.css" -- lowercase s, matching none
git ls-files 'src/MVC5/MvcMusicStore/Content/*' 'src/MVC5/MvcMusicStore/Scripts/*' \
             'src/MVC5/MvcMusicStore/Images/*'  'src/MVC5/MvcMusicStore/fonts/*' | wc -l   # -> 27

# --- Section 8.7  One Json() return in the whole migration source -------------
git grep -n 'Json(' -- 'src/MVC5/MvcMusicStore/Controllers/*.cs'
# -> ShoppingCartController.cs:83:  return Json(results);

# --- Section 4.6  Which navigations EF 6 could lazy-load, and which not -------
git grep -n 'virtual' -- 'src/MVC5/MvcMusicStore/Models/Album.cs' \
                         'src/MVC5/MvcMusicStore/Models/Cart.cs'
git grep -n 'List<Album> Albums' -- 'src/MVC5/MvcMusicStore/Models/Genre.cs'
# -> Album.cs:30 Genre, :31 Artist, :32 OrderDetails and Cart.cs:17 Album are virtual
#    Genre.cs:10 Albums is NOT -- which is why genre browse had to eager-load

# --- Section 9.1  The legacy-directive census: 19 of 27, not 17 ---------------
python - <<'PY'
import subprocess, re
files = subprocess.run(['git','ls-files','src/MVC5/MvcMusicStore/*.cs'],
                       capture_output=True, text=True).stdout.split()
LEG = re.compile(r'^\s*using\s+(System\.Web|System\.Data\.Entity|System\.Configuration'
                 r'|Owin|Microsoft\.Owin|Microsoft\.AspNet\.Identity)\b')
naive = tol = boms = 0; without = []
for f in files:
    raw = open(f, 'rb').read()
    boms += raw[:3] == b'\xef\xbb\xbf'
    n = any(LEG.match(l) for l in raw.decode('utf-8').splitlines())        # BOM breaks line 1
    t = any(LEG.match(l) for l in raw.decode('utf-8-sig').splitlines())    # BOM stripped
    naive += n; tol += t
    if not t: without.append(f.split('MvcMusicStore/')[-1])
print('files=%d  utf8-BOM=%d  naive=%d  bom_tolerant=%d' % (len(files), boms, naive, tol))
print('the 8 with no legacy directive:'); [print('  ', w) for w in without]
PY
# -> files=27  utf8-BOM=27  naive=17  bom_tolerant=19
#    the 8: Models/AccountViewModels.cs, Models/Album.cs, Models/Cart.cs,
#           Models/Genre.cs, Models/OrderDetail.cs, Properties/AssemblyInfo.cs,
#           ViewModels/ShoppingCartRemoveViewModel.cs, ViewModels/ShoppingCartViewModel.cs
# Per-directive file counts, same BOM-tolerant method:
#    System.Web.Mvc 11 | System.Web 11 | Microsoft.Owin* 4 | Microsoft.AspNet.Identity* 4
#    System.Data.Entity 3 | Owin 3 | System.Web.Routing 2 | System.Web.Optimization 2
#    System.Configuration 1
# NOTE: `using System.Web;` is 11 files, not 10 -- Models/SampleData.cs is the eleventh.
# Where a count in another input disagrees, the repository governs (section 1.3).

# --- Section 12.1  No test of any kind, by two independent methods ------------
git ls-files | grep -ic test                                                  # -> 0
git grep -l -E "TestClass|\[Fact\]|\[Theory\]|xunit|NUnit|MSTest" \
         -- '*.cs' '*.csproj' '*.config' | wc -l                              # -> 0

# --- Section 12.3  The legacy baseline needs both database pairs --------------
git ls-files 'src/MVC5/MvcMusicStore/App_Data/*'
# -> MvcMusicStore.mdf / MvcMusicStore_log.ldf                       (catalog)
#    aspnet-MvcMusicStore-20131025034205.mdf / ..._log.ldf           (Identity)

# --- Section 14.2  The constraint this work was held to -----------------------
git status --porcelain
# -> only new files under docs/modernization/
```

---

*Deliverable 05 of 13. Owns the cutover decision and its accepted losses. Every claim about the
existing system above carries an inline file location; every repository-wide count is reproducible by
the command stated beside it or in Appendix A; every one of deliverable 12's twenty-two blockers has a
named resolution in section [13](#13-resolution-register--all-22-blockers-of-deliverable-12). No user
rules were provided for this project. No repository file outside `docs/modernization/` was created,
modified or deleted in its production, and nothing in it authorizes a code change before the
assessment is approved.*
