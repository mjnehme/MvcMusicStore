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
instructions independently restate the same gate. Nothing in this repository was created, modified or
deleted to produce this document, and **every construct named below stays exactly where it is**. The
final acceptance check for this whole assessment is that `git status --porcelain` shows nothing but new
files under `docs/modernization/`.

This constraint bites hardest here, because this document reads like a work order. It is not one. It is a
list a reader approves *before* anyone touches a file. Two of the entries below — the mutating `GET` at
[F-12-07](#f-12-07--the-synchronous-tryupdatemodel-and-the-class-level-bind-attribute) and the path
casing at [F-12-17](#f-12-17--filesystem-path-casing) — would each take a one-line edit to fix, and
neither was fixed.

### 1.4 Authoring contract, and the absence of user rules

**No user-specified rules were provided for this project.** `review_rules` returns exactly *"No user
rules provided."*, verified directly during this work. There is accordingly no rule to name, summarize or
cite, and no file forced into scope by one. The absence is not a licence to lower the bar; enterprise
best practice applies instead, and this document holds itself to four explicit contracts:

1. **Every entry carries a file location.** Citations are inline as `[<path>:<locator>]` at the point the
   claim is made, never collected in a list at the end. Paths are repository-relative and resolve in the
   checkout. A citation with no locator does not satisfy this contract.
2. **Every entry states which edition or editions it holds in.** "The application" is three
   applications, and a blocker in one is not a blocker in the others. MVC 3's SQL Server Compact
   dependency is MVC 3's alone; the OWIN constructs are MVC 5's alone; `TryUpdateModel` is in all three.
3. **Repository-wide claims carry their reproducing command.** A count or an absence has no single line
   to point at, so the evidence is the command, printed beside the claim and collected in
   [section 8](#8-reproducibility-appendix). That is the stronger form of evidence, because a reader can
   re-run it.
4. **One fact, one owner.** Where a fact belongs to another deliverable, this document cites it and does
   not restate it. Section 1.5 lists the owners.

### 1.5 What this document does not own

| Decision or fact | Owner | What this document does instead |
| --- | --- | --- |
| **Target framework, SDK band, project-format conversion** | [04](04-dotnet8-migration-strategy.md) | Names no framework version and no package version. Entries state that a construct has no successor *in ASP.NET Core*, which is true of every candidate target. |
| **How each blocker is resolved** — pipeline, DI, configuration, EF Core, Identity, static assets, cutover | [05](05-aspnet-core-migration-approach.md) | Enumerates and locates. Where a successor construct is named, it is named factually; the design is 05's, and where 05 has a genuine choice this document records that the choice exists rather than making it. |
| **Hosting target, deployment model, telemetry mechanism, data-protection key store** | [06](06-azure-hosting-recommendations.md) | Cross-references only. The Linux path-casing audit at F-12-17 is 06's requirement, not this document's recommendation. |
| **Per-edition build outcomes** | [10](10-build-and-deployment-requirements.md) | Cites 10 for every build result, including MVC 5's verified build and MVC 4's committed-configuration failure. The diagnosis is **not** restated. |
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
toolchain produces for free. These entries are numerous and some are laborious, but **none of them is
dangerous**, because none of them can reach production undetected.

**Group two — the successor exists, and its default behaviour differs. These compile, deploy, and then
behave differently.** The code is valid in the target. It builds with zero warnings. It starts. Then a
navigation property is null where it used to be populated, a JSON field arrives camel-cased where the
JavaScript expects PascalCase, a stylesheet 404s on a case-sensitive filesystem, or a `Dispose` override
disposes an object the container still owns. **Nothing announces any of it.**

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

Twenty-two blockers. Fourteen fail at compile time; eight are silent.

| ID | Blocker | Editions | Failure mode |
| --- | --- | --- | --- |
| [F-12-01](#f-12-01--sql-server-compact-40-as-the-catalogue-provider) | SQL Server Compact 4.0 as the catalogue provider | MVC 3 | Compile-time |
| [F-12-02](#f-12-02--systemweboptimization-bundling) | `System.Web.Optimization` bundling, with `{version}` and glob tokens | MVC 5, MVC 4 | Compile-time |
| [F-12-03](#f-12-03--the-katana-iappbuilder-abstraction-and-the-owin-startup-attribute) | The Katana `IAppBuilder` abstraction and the OWIN startup attribute | MVC 5 | Compile-time |
| [F-12-04](#f-12-04--systemwebhttpapplication-and-the-globalasax-markup-declaration) | `System.Web.HttpApplication` and the `Global.asax` markup declaration | all three | Compile-time |
| [F-12-05](#f-12-05--handleerrorattribute--the-entire-error-handling-policy) | `HandleErrorAttribute` — the entire error-handling policy | MVC 5, MVC 4 | Compile-time |
| [F-12-06](#f-12-06--systemwebmvchandleerrorinfo-as-a-view-model) | `System.Web.Mvc.HandleErrorInfo` as a view model | MVC 5 | Compile-time |
| [F-12-07](#f-12-07--the-synchronous-tryupdatemodel-and-the-class-level-bind-attribute) | The synchronous `TryUpdateModel` and the class-level `[Bind]` | all three | Compile-time |
| [F-12-08](#f-12-08--three-httpnotfound-calls) | Three `HttpNotFound()` calls | MVC 5 | Compile-time |
| [F-12-09](#f-12-09--challengeresult-and-its-executeresult-override) | `ChallengeResult` and its `ExecuteResult` override | MVC 5 | Compile-time |
| [F-12-10](#f-12-10--the-blockviewhandler-mapping-and-the-razor-host-section-group) | The `BlockViewHandler` mapping and the Razor host section group | MVC 5 | Compile-time |
| [F-12-11](#f-12-11--the-axd-ignore-route) | The `.axd` ignore route | MVC 5 | Compile-time |
| [F-12-12](#f-12-12--assembly-metadata-in-a-source-file) | Assembly metadata in a source file | all three | Compile-time |
| [F-12-13](#f-12-13--windows-authentication-connection-strings-and-file-attached-databases) | Windows-authentication connection strings and file-attached databases | all three | Compile-time |
| [F-12-14](#f-12-14--mvc-4s-committed-build-configuration) | MVC 4's committed build configuration | MVC 4 | Compile-time |
| [F-12-15](#f-12-15--lazy-loading-is-on-by-default-in-ef-6-and-off-in-ef-core) | Lazy loading is on by default in EF 6 and off in EF Core | all three | **Silent runtime** |
| [F-12-16](#f-12-16--json-property-naming-flips-to-camelcase) | JSON property naming flips to camelCase | all three | **Silent runtime** |
| [F-12-17](#f-12-17--filesystem-path-casing) | Filesystem path casing | MVC 5 | **Silent runtime** |
| [F-12-18](#f-12-18--a-framework-version-mismatch-inside-mvc-5s-own-configuration) | A framework-version mismatch inside MVC 5's own configuration | MVC 5 | **Silent runtime** |
| [F-12-19](#f-12-19--connection-string-resolution-by-convention) | Connection-string resolution by convention, two conventions | MVC 5 | **Silent runtime** |
| [F-12-20](#f-12-20--dispose-overrides-on-objects-a-container-will-own) | `Dispose` overrides on objects a container will own | all three | **Silent runtime** |
| [F-12-21](#f-12-21--the-identity-schema-is-not-knowable-from-the-repository) | The Identity schema is not knowable from the repository | MVC 5 | **Silent runtime** |
| [F-12-22](#f-12-22--no-usable-schema-baseline-exists) | No usable schema baseline exists | MVC 5, MVC 4 | **Silent runtime** |

---

## 3. Group one — no successor, or the API itself is gone (compile-time)

### F-12-01 — SQL Server Compact 4.0 as the catalogue provider

**Editions:** MVC 3 only.

`web.config` declares the catalogue connection as `Data Source=|DataDirectory|MvcMusicStore.sdf` with
`providerName="System.Data.SqlServerCe.4.0"`
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:56-58]. That is the **only** connection string
the file declares — there is no second entry, which is the same evidence
[09 §5.1](09-security-assessment.md) uses to establish that MVC 3's credential store is inherited from
machine configuration rather than declared.

Two facts make this a hard stop rather than a provider swap:

- **No `.sdf` exists anywhere in the checkout.** `git ls-files '*.sdf' | wc -l` returns `0`. The database
  the connection string names has never been committed, so the file is created on first run by a provider
  that must already be installed machine-wide. What *is* committed under the tutorial assets is
  `src/MVC3/MvcMusicStore-Assets/Data/MvcMusicStore.mdf` — a SQL Server file for a **different engine**
  than the one the application's own configuration names.
- **The data layer is bound to the EF 4.1 generation.** The project references `EntityFramework`
  4.1.0.0 by `HintPath` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:37-40] and
  the framework assembly `System.Data.Entity` [:41], and nothing else.

**Failure mode: compile-time.** The provider assembly `System.Data.SqlServerCe.4.0` has no build for the
target framework, so the reference cannot resolve. A community EF Core provider for SQL Server Compact
exists but was **abandoned at the EF Core 2.2 generation**, so there is no supported provider for any
current target either — this is not a version-lag problem that waiting solves.

**Successor: none for the engine.** MVC 3's data layer cannot be ported without **re-targeting the
provider outright**, which means choosing a different database engine, not upgrading a package.
[02 §4.1](02-dependency-inventory.md) records the provider as an undeclared machine-wide dependency and
[09 §5.8](09-security-assessment.md) records that it is out of support and receives no security
servicing. [10 §10.2](10-build-and-deployment-requirements.md) owns the two-engine topology this
produces — MVC 3 needs SQL Server Compact for the catalogue **and** a SQL Server instance for
credentials, simultaneously.

**Owner:** [10](10-build-and-deployment-requirements.md) for the topology; the triage decision that MVC 3
is not a migration source is [03](03-modernization-roadmap.md)'s to sequence.

### F-12-02 — `System.Web.Optimization` bundling

**Editions:** MVC 5 and MVC 4 (MVC 3 has no `App_Start` folder and no bundling implementation at all —
see [01 §3.6](01-architecture-overview.md)).

Five bundles are registered in the migration source
[src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs:11,:14,:19,:22,:26], and the registrations use two
token forms that are the actual obstacle rather than the bundling itself:

- a **`{version}` token** — `"~/Scripts/jquery-{version}.js"` [:12], which resolves a version number out
  of the filename at runtime;
- **glob tokens** — `"~/Scripts/jquery.validate*"` [:15] and `"~/Scripts/modernizr-*"` [:20].

**Failure mode: compile-time.** `System.Web.Optimization` is a package with no successor package;
`ScriptBundle`, `StyleBundle`, `BundleCollection` and `BundleTable` do not exist in the target. The
registration call in `Application_Start`
[src/MVC5/MvcMusicStore/Global.asax.cs:18] goes with it, and the namespace registered for views
[src/MVC5/MvcMusicStore/Views/Web.config:18] becomes meaningless.

**The view surface is larger than the registration surface.** Eleven call sites depend on the framework
entirely — **10** `@Scripts.Render` and **1** `@Styles.Render`, the latter in the shared layout
[src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:7], with three of the ten in the same file
[:8], [:41], [:42] and the remaining seven spread across the Account, Checkout and StoreManager views.
Verify with `git grep -n '@Scripts.Render' -- 'src/MVC5/*.cshtml' | wc -l` → `10` and the same for
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
`app.UseExternalSignInCookie(...)` [:20] are Katana extensions, not middleware registrations of the kind
the target uses.

**Successor: the responsibilities survive, the abstraction does not.** Cookie authentication is a
first-class framework feature in the target, so the *capability* at [:14-18] carries over while the
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
[:13-21], and they do not share one fate — which is why this entry enumerates them rather than treating
the file as a single unit:

| Registration | Line | Disposition |
| --- | --- | --- |
| `AreaRegistration.RegisterAllAreas()` | [:15] | Dead scaffolding — there is no `Areas` folder ([08 §9.1](08-technical-debt-register.md)) |
| `FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters)` | [:16] | Its one filter has no successor — [F-12-05](#f-12-05--handleerrorattribute--the-entire-error-handling-policy) |
| `RouteConfig.RegisterRoutes(RouteTable.Routes)` | [:17] | Has a direct successor form — [F-12-11](#f-12-11--the-axd-ignore-route) |
| `BundleConfig.RegisterBundles(BundleTable.Bundles)` | [:18] | No successor — [F-12-02](#f-12-02--systemweboptimization-bundling) |
| `System.Data.Entity.Database.SetInitializer(new SampleData())` | [:20] | Replaced by explicit schema management; note it is registered a **second** time at [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:16], which [08 §5.1](08-technical-debt-register.md) owns |

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

**Editions:** MVC 5 and MVC 4, identically.

`filters.Add(new HandleErrorAttribute());`
[src/MVC5/MvcMusicStore/App_Start/FilterConfig.cs:10], and the same single line at
[src/MVC4/MvcMusicStore/App_Start/FilterConfig.cs:10].

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
when custom errors are enabled, and **no edition enables them** — all 24 occurrences of `<customErrors>`
sit inside commented template blocks in the six transform files and none appears in any live
`Web.config` ([09 §6.10](09-security-assessment.md) owns this). The attribute is therefore closer to
inert than active in the shipped configuration, which means the target's behaviour is being *chosen*
rather than *preserved*, and the difference must be deliberate rather than inherited.

**Failure mode: compile-time.** The type is absent from the target; `GlobalFilterCollection` is absent
with it.

**Successor: exception-handling middleware**, which is a pipeline registration rather than a filter, plus
whatever policy 05 decides for status-code responses and for what may be disclosed.

**Owner:** [05](05-aspnet-core-migration-approach.md).

### F-12-06 — `System.Web.Mvc.HandleErrorInfo` as a view model

**Editions:** MVC 5.

The shared error view declares `@model System.Web.Mvc.HandleErrorInfo`
[src/MVC5/MvcMusicStore/Views/Shared/Error.cshtml:1]. The rest of the file is two static headings
[:7-8] — it renders **no property of that model at all**.

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
`Order` instance constructed one line earlier [:28], inside the checkout POST action whose parameter is a
raw `FormCollection` [:26].

What makes the call safe today is a **separate** construct: a class-level attribute on the entity,
`[Bind(Include = "FirstName,LastName,Address,City,State,PostalCode,Country,Phone,Email")]`
[src/MVC5/MvcMusicStore/Models/Order.cs:8]. That include list is the entire over-posting control at
checkout — [09 §6.4](09-security-assessment.md) owns that consequence — and both halves of the mechanism
have to move together.

**Failure mode: compile-time, twice over.** The target exposes **only** the asynchronous form,
`TryUpdateModelAsync`; there is no synchronous overload, so the call at [:29] does not compile. And
`[Bind(Include = ...)]` is superseded by explicit binding models, so the attribute at [Order.cs:8] has no
target either. Neither is a namespace substitution: the method signature changes and the attribute
disappears. `Order.cs` also carries `using System.Web.Mvc` [:4] purely to reference that attribute — a
model-layer file with an MVC dependency, which is why the port of this file is not confined to
`Controllers/`.

**Successor: an explicit input model — and it must carry TEN properties, not nine.** This is the detail
most likely to be lost, because nine is the number written down in the repository and ten is the number
the action actually reads:

- **Nine** come from the `[Bind]` include list [Order.cs:8].
- **The tenth is `PromoCode`**, which the action reads **separately, straight out of the raw form** —
  `values["PromoCode"]` [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:33], compared
  case-insensitively against `const string PromoCode = "FREE"` [:12], with the form collection arriving
  as the action parameter [:26]. It is not on the `[Bind]` list because it is not bound at all.
- **`PromoCode` belongs on the input model and *not* on the `Order` entity.** `Order` has no such
  property. Its whole surface is fourteen properties: the nine the bind list names, four suppressed with
  `[ScaffoldColumn(false)]` — `OrderId` [Order.cs:11-12], `OrderDate` [:14-15], `Username` [:17-18] and
  `Total` [:63-64] — and the `OrderDetails` navigation collection [:66]. Adding a tenth persisted property
  would put a transient form value on the entity.

A plan that carries nine properties compiles, binds and persists correctly, and **silently loses
promo-code handling** — which in this application means the order-completion branch [:38-55] is
unreachable and every checkout returns to the form at [:36]. The nine-versus-ten fact is recorded here
because it is a property of the *current* code that no automated tool infers;
[09 §6.4](09-security-assessment.md) states the same ten and defers the model to 05.

**Owner:** [05](05-aspnet-core-migration-approach.md), which owns the input model and its tests — valid,
invalid and missing promo-code values.

### F-12-08 — Three `HttpNotFound()` calls

**Editions:** MVC 5.

Three calls, all in the administration controller, all in the same shape — a `Find` that may return
`null`, a null check, and a not-found result:
[src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:35] (Details),
[:76] (Edit) and [:108] (Delete). `git grep -n 'HttpNotFound()' -- 'src/MVC5/*.cs' | wc -l` returns
exactly `3`, so this is a census rather than a sample.

**Failure mode: compile-time.** `HttpNotFound()` is a `System.Web.Mvc.Controller` helper returning
`HttpNotFoundResult`; **neither the method nor the return type exists** on the target's controller base
class. The call does not compile.

**Successor: `NotFound()`**, returning the target's `NotFoundResult`. This is the closest thing in the
document to a mechanical rename, and it is listed because a census that omits the easy items is not a
census — 05 needs the count to be right, not interesting.

**One related site is deliberately *not* in this entry.** The fourth `Find` in the same controller,
`DeleteConfirmed`, has **no** null check at all: it passes the result straight to `Albums.Remove`
[:119-120]. That compiles in the target exactly as it compiles today and throws exactly as it throws
today, so it is not a migration blocker; [08 §5.4](08-technical-debt-register.md) owns it as debt. Three
of four call sites guarded, one not, is a distribution worth not smoothing over.

**Owner:** [05](05-aspnet-core-migration-approach.md).

### F-12-09 — `ChallengeResult` and its `ExecuteResult` override

**Editions:** MVC 5.

`AccountController` declares a private nested action result to issue an OWIN authentication challenge:
`private class ChallengeResult : HttpUnauthorizedResult`
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:394], which overrides
`public override void ExecuteResult(ControllerContext context)` [:411] and inside it calls
`context.HttpContext.GetOwinContext().Authentication.Challenge(properties, LoginProvider)` [:418].

**Failure mode: compile-time, and every layer of it fails independently.** There are four separate
breaks in one twenty-seven-line class, which is why it cannot be ported by adjusting a namespace:

1. **The base type is gone.** `HttpUnauthorizedResult` does not exist in the target.
2. **The override signature is gone.** Action results in the target implement
   `ExecuteResultAsync(ActionContext)` — the method name, its return type and its parameter type all
   differ, so `override` cannot bind.
3. **The challenge mechanism is gone.** `GetOwinContext()` is a Katana extension over `System.Web`
   ([F-12-03](#f-12-03--the-katana-iappbuilder-abstraction-and-the-owin-startup-attribute)), and
   `IAuthenticationManager` — held as a property at [:338-341] — has no successor type.
4. **`using Microsoft.Owin.Security;`** [:10] supports `AuthenticationProperties` at [:413], and goes
   with the rest of the OWIN surface.

**05 has a genuine choice here, and this document does not make it.** Every external-provider
registration in the application is **commented out** — Microsoft Account, Twitter, Facebook and Google,
all four inert [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:23-35] — so the external-login surface
this class exists to serve is scaffolded but disabled
([08 §9.3](08-technical-debt-register.md), and [09 §6.11](09-security-assessment.md) records it as
deployed attack surface). That means 05 may either map the class onto the framework's external-challenge
flow, **or remove the disabled external-login surface entirely** and delete the type along with the
views that render it. **Recorded, not decided:** the choice exists, it is 05's, and either way this is a
rewrite rather than a port, with route and behaviour tests attached.

**Successor: the framework's authentication challenge**, if the surface is retained; nothing, if it is
removed.

**Owner:** [05](05-aspnet-core-migration-approach.md).

### F-12-10 — The `BlockViewHandler` mapping and the Razor host section group

**Editions:** MVC 5 (MVC 4 and MVC 3 ship the same file shape with their own versions).

`Views/Web.config` carries two distinct constructs, and both are properties of the IIS integrated
pipeline rather than of the application:

- **A handler that makes the views directory unservable.** `<remove name="BlockViewHandler"/>` followed by
  `<add name="BlockViewHandler" path="*" verb="*" preCondition="integratedMode"
  type="System.Web.HttpNotFoundHandler" />` [src/MVC5/MvcMusicStore/Views/Web.config:31-32] — every path,
  every verb, mapped to a not-found handler so that `.cshtml` files under `Views/` cannot be requested
  directly.
- **A Razor host and page-base-type registration.** A `configSections` section group declaring the
  `host` and `pages` sections [:5-8], then the `system.web.webPages.razor` element that sets
  `factoryType` to `MvcWebRazorHostFactory` [:12] and `pageBaseType` to `System.Web.Mvc.WebViewPage`
  [:13], with **six** namespaces imported into every view [:14-21] — `System.Web.Mvc`,
  `System.Web.Mvc.Ajax`, `System.Web.Mvc.Html`, `System.Web.Optimization`, `System.Web.Routing` and
  `MvcMusicStore` [:15-20].

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

**Editions:** MVC 5.

`routes.IgnoreRoute("{resource}.axd/{*pathInfo}");`
[src/MVC5/MvcMusicStore/App_Start/RouteConfig.cs:14].

**Failure mode: compile-time.** `IgnoreRoute` is a `RouteCollection` extension in `System.Web.Routing`,
and neither the extension nor the collection type exists in the target.

**Successor: none; dropped.** `.axd` handlers were ASP.NET's mechanism for serving framework resources
such as `WebResource.axd` and `ScriptResource.axd`. The target has no such handler, so there is nothing
for the route to exclude — the line is not replaced, it becomes unnecessary.

**The route it accompanies is the favourable half of this entry.** The single conventional route
[:16-20] — `{controller}/{action}/{id}` with defaults `Home`, `Index` and `UrlParameter.Optional` — has a
**direct successor form**, and it is the whole of the application's URL surface: one route, no areas, no
attribute routing, no constraints. [01 §4.1](01-architecture-overview.md) owns the routing description.

**Owner:** [05](05-aspnet-core-migration-approach.md).

### F-12-12 — Assembly metadata in a source file

**Editions:** all three — `git ls-files '*AssemblyInfo.cs'` returns three paths, one per edition.

The migration source carries **12** assembly-level attributes in a source file: title, description,
configuration, company, product and copyright
[src/MVC5/MvcMusicStore/Properties/AssemblyInfo.cs:8-13], COM visibility [:20], the type-library GUID
[:23], and the assembly and file versions [:34-35].

**Failure mode: compile-time**, and it is the mildest in this group: the SDK project format generates
these attributes from build properties, so a hand-written file that declares the same attributes produces
**duplicate-attribute** compile errors unless the file is removed or generation is switched off.

**Successor: MSBuild properties in the project file.** The metadata survives; the file does not.

**Owner:** [04](04-dotnet8-migration-strategy.md), which owns the project-format conversion.

### F-12-13 — Windows-authentication connection strings and file-attached databases

**Editions:** all three, with MVC 4 the worst case.

MVC 5 declares two connection strings, and both are unusable against a managed SQL service as written:
`Data Source=(LocalDb)\MSSQLLocalDB;AttachDbFilename=|DataDirectory|\aspnet-MvcMusicStore-20131025034205.mdf;Initial Catalog=...;Integrated Security=True`
[src/MVC5/MvcMusicStore/Web.config:12] and the catalogue equivalent with
`AttachDbFilename=|DataDirectory|\MvcMusicStore.mdf` and `Integrated Security=True` [:13].

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
`(LocalDb)\v11.0` [src/MVC4/MvcMusicStore/Web.config:13] and `(LocalDB)\v11.0` [:19] — SQL Server 2012
LocalDB — while **its own README documents `(LocalDB)\MSSQLLocalDB`** [src/MVC4/README.md:45], and Visual
Studio 2022, the README's stated prerequisite, installs no v11.0 instance. The repository therefore
disagrees with itself about which engine the application needs, and neither statement can be assumed
correct without a host check. [10](10-build-and-deployment-requirements.md) owns the per-edition topology
and the consequence for running MVC 4.

**Failure mode: compile-time**, though by a different route than the rest of this group: the connection
strings live in a configuration file the target does not read at all. `Web.config` `connectionStrings` and
`appSettings` are not a configuration source in the target, and `ConfigurationManager.AppSettings` — read
directly from startup code [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:23-24] via
`using System.Configuration;` [:7] — does not resolve. The *values* must be re-expressed **and** the
*mechanism* that reads them must be replaced.

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
committed project and solution files, and 10 §13.1 records that MVC 4 **fails as committed**, in both
solutions, before compilation, and builds only under host-side command-line overrides.

The locations being classified, for reference only — 10 owns what is wrong with each and why. The two
project-file defects: an **unconditional** NuGet target import,
`<Import Project="$(SolutionDir)\.nuget\nuget.targets" />` carrying no `Condition` attribute
[src/MVC4/MvcMusicStore/MvcMusicStore.csproj:360], and the **24** package `HintPath` entries that resolve
one directory above the committed payload, the first at
[src/MVC4/MvcMusicStore/MvcMusicStore.csproj:66]. Then, separately, the stale fourth solution file, whose
declared project path resolves to a directory that does not exist
[src/MVC4/MvcMusicStore/MvcMusicStore.sln:4].

**The classification.** MVC 4 cannot serve as a migration source or as an executable behavioural
baseline in the state the repository ships, because a source you cannot build from its own committed
configuration cannot be diffed against a port at runtime. That is a blocker on *using* MVC 4, not on
porting it — and since MVC 4 is not the migration source, its practical effect is narrower than it looks:
it removes MVC 4 from the set of candidate runtime baselines and leaves MVC 5, whose build **is** verified
([10 §5.4](10-build-and-deployment-requirements.md): restore and build both succeed, Debug and Release,
zero warnings and zero errors).

**Failure mode: compile-time**, by definition — the build fails before any compilation happens.

**Successor: not applicable.** MVC 4 is retained read-only as a historical reference. Nothing here is
repaired: the environment setup instructions explicitly direct that these defects be documented and
worked around at the command line rather than fixed in the repository, which is what
[§1.3](#13-the-no-modification-constraint) requires anyway.

**Owner:** [10](10-build-and-deployment-requirements.md) for the diagnosis and the workaround;
[08 §8.4](08-technical-debt-register.md) for severity and owner.

---

## 4. Group two — the successor exists and its default differs (silent breakage)

Everything in this section **compiles**. That is the property that makes it dangerous, and it is why each
entry enumerates sites rather than concepts.

### F-12-15 — Lazy loading is on by default in EF 6 and off in EF Core

**Editions:** all three (the site census below is MVC 5, the migration source).

EF 6 populates navigation properties on demand through runtime proxies. EF Core does not: an
unloaded reference navigation is simply `null`. The code that depends on this compiles identically under
both.

**The mechanism is precise, and it explains which sites are affected and which are not.** EF 6 lazy
loading requires the navigation property to be `virtual`, and in this model they are not uniformly so:

| Navigation | Declared | Lazy-loadable under EF 6 |
| --- | --- | --- |
| `Album.Genre` | `virtual` [src/MVC5/MvcMusicStore/Models/Album.cs:30] | Yes |
| `Album.Artist` | `virtual` [src/MVC5/MvcMusicStore/Models/Album.cs:31] | Yes |
| `Album.OrderDetails` | `virtual` [src/MVC5/MvcMusicStore/Models/Album.cs:32] | Yes |
| `Cart.Album` | `virtual` [src/MVC5/MvcMusicStore/Models/Cart.cs:17] | Yes |
| `Genre.Albums` | **not** `virtual` [src/MVC5/MvcMusicStore/Models/Genre.cs:10] | **No** |

That table is the reason the affected set is exactly what it is. Every silent-failure site below reaches
through one of the three `virtual` navigations, and the one non-`virtual` collection navigation is
**already eager-loaded out of necessity** — `storeDB.Genres.Include("Albums")`
[src/MVC5/MvcMusicStore/Controllers/StoreController.cs:30] is not an optimization, it is mandatory, which
is why the genre-browse page is unaffected.

**Three mechanisms, not one.** A flat list of "navigation dereferences" conflates cases that need
opposite treatment, so the census separates them:

**(a) Client-side dereference after materialization — definite break.** The entity is materialized first,
then the navigation is read in C# or in a view. EF Core returns `null`, so the read throws a null
reference or renders empty. **Six sites:**

| # | Query (no `Include`) | Dereference | Effect in the target |
| --- | --- | --- | --- |
| 1 | `storeDB.Albums.Find(id)` [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:38] | `@Model.Genre.Name` [src/MVC5/MvcMusicStore/Views/Store/Details.cshtml:16] and `@Model.Artist.Name` [:20] | Album detail page throws |
| 2 | `db.Albums.Find(id)` [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:32] | `@Html.DisplayFor(model => model.Artist.Name)` [src/MVC5/MvcMusicStore/Views/StoreManager/Details.cshtml:18] and `model.Genre.Name` [:26] | Admin detail page renders empty or throws |
| 3 | `storeDB.Carts.Single(item => item.RecordId == id)` [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:61-62] | `.Album.Title` on the same line [:62] | Cart removal throws before it responds |
| 4 | `cart.GetCartItems()` — `ToList()` with no `Include` [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:100] | `.Select(a => a.Album.Title)` [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:91-92] | Cart summary in the shared layout throws — on **every** page, since it is a layout-level child action |
| 5 | `cart.GetCartItems()` via the cart view model [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:17-24] | `item.Album.Title` [src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:64] and `@item.Album.Price` [:68] | Cart page throws |
| 6 | `GetCartItems()` inside order creation [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:129] | `item.Album.Price` [:145] | **Order total silently wrong or checkout throws** |

Site 6 deserves its own sentence, because it is the one with a financial consequence and it sits three
lines from a site that is *not* affected. The same loop explicitly loads the album for the order-detail
row — `var album = _db.Albums.Find(item.AlbumId);` [:134], used for `UnitPrice` at [:140] — and then
computes the running order total from the **lazy** navigation instead [:145]. One loop, two ways of
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

**(c) Nested collection aggregates — translation risk, and the failure is louder than a null.** Two
sites: the genre menu orders by a nested `Sum` over each genre's albums' order-detail quantities
[src/MVC5/MvcMusicStore/Controllers/StoreController.cs:46-52], and the home page orders by
`a.OrderDetails.Count()` [src/MVC5/MvcMusicStore/Controllers/HomeController.cs:29-32]. EF 6 will silently
evaluate parts of such queries on the client when it cannot translate them; **EF Core throws** rather
than silently client-evaluating. So the risk here is not a wrong answer, it is an
`InvalidOperationException` at a query that used to work — and the genre menu, like site 4 above, renders
on every page from the layout.

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
`return Json(results);` [:83].

The page's JavaScript reads **exactly those PascalCase names**, five times:
`data.ItemCount` [src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:21], `data.DeleteId` [:22],
`data.ItemCount` again [:24], `data.CartTotal` [:27], `data.Message` [:28] and `data.CartCount` [:29].

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
still `200`, the JSON is still well-formed — and every one of the five reads above evaluates to
`undefined`. The row does not fade out, the count does not update, the total does not change, and no
error appears in any log, because no error occurs.

**The transport constrains the fix, so it is recorded here.** The removal posts through
`$.post(PostToUrl, { "id": recordToDelete }, ...)`
[src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:17] — **with no surrounding form**: the trigger is
a plain anchor carrying `data-id` and `data-url` attributes [:74-75]. Because no `contentType` and no
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

**Editions:** MVC 5.

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
`<httpRuntime targetFramework="4.5"/>` [:34].

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

**Editions:** MVC 5.

Two `DbContext` classes, **two different conventions, neither of which EF Core honours**:

- `MusicStoreEntities : DbContext` declares **no constructor at all** — six `DbSet` properties and
  nothing else [src/MVC5/MvcMusicStore/Models/MusicStoreEntities.cs:5-13]. EF 6 resolves its connection
  string by matching the **class name** against a `connectionStrings` entry, and the matching entry is
  named `MusicStoreEntities` [src/MVC5/MvcMusicStore/Web.config:13]. The coupling between the class and
  its database is a name, expressed nowhere in the code.
- `ApplicationDbContext : IdentityDbContext<ApplicationUser>` passes the name **explicitly**,
  `: base("DefaultConnection")` [src/MVC5/MvcMusicStore/Models/IdentityModels.cs:12-13], matching the
  entry at [src/MVC5/MvcMusicStore/Web.config:12].

**Failure mode: silent runtime, and the first context is the dangerous one.** A `DbContext` subclass with
no constructor is perfectly valid in EF Core — it compiles, and it can be instantiated. What changes is
that nothing tells it where the database is: EF Core has no class-name convention, and no `Web.config` to
consult ([F-12-13](#f-12-13--windows-authentication-connection-strings-and-file-attached-databases)). The
class-name coupling does not fail to compile, it fails to *mean anything*, and because the code that
loses meaning is the **absence** of a constructor, there is nothing at that location for a reviewer to
notice. A reader scanning `MusicStoreEntities.cs` for migration work sees thirteen lines of `DbSet`
declarations and no problem.

The second context is less risky for the opposite reason: `base("DefaultConnection")` is a constructor
call to a base overload that no longer exists, so it fails at compile time and announces itself. Two
files, the same underlying change, opposite failure modes — which is why they are one entry rather than
two.

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
| [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:125-128] | `MusicStoreEntities` | field initializer [:15] |
| [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:125-128] | `MusicStoreEntities` | field initializer |
| [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:112-115] | `MusicStoreEntities` | field initializer |
| [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:324-331] | `UserManager<ApplicationUser>` | chained constructor [:19] |

Today each is correct: the controller creates the object, so the controller disposes it. The MVC 5 account
controller is even careful about it — it null-guards and nulls the field [:326-330].

**Failure mode: silent runtime, and the mechanism is worth stating exactly.** Once the container owns
these lifetimes, the object handed to the controller is **scoped to the request**, not to the controller,
and the container disposes it at the end of the scope. An override that disposes it earlier compiles
cleanly — `Dispose(bool)` still exists on the target's controller base class, so there is no signature
error and no warning — and then produces `ObjectDisposedException` at whatever code path touches the
context after the controller is released. Two properties make this hard to catch: the failure is
**order-dependent**, so it may not reproduce under a single-request test, and it surfaces at a call site
that has nothing wrong with it.

The account controller carries the sharper version of the risk, because its disposal is entangled with
[F-12-09](#f-12-09--challengeresult-and-its-executeresult-override)'s rewrite: the `UserManager` it
disposes is one it builds by hand through a chained constructor that also builds a `UserStore` and an
`ApplicationDbContext` [:19]. When the container supplies all three, an override disposing the middle one
reaches an object two other consumers may still hold.

**Successor: remove the overrides.** There is no replacement construct — container-owned lifetimes need no
disposal from the consumer.

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
[src/MVC5/MvcMusicStore/packages.config:8] and `Microsoft.AspNet.Identity.EntityFramework` 1.0.0 [:9] —
whose schema predates all six of those columns. Two independent lines of evidence pointing the same way
is a strong prior. It is still a prior. [09](09-security-assessment.md) reaches the same conclusion by the
same route under finding **F-09-03** and qualifies it identically; this document's run corroborates it
rather than adding a second claim.

**Failure mode: silent runtime.** An EF Core initial migration generated from the ported Identity model
creates the *modern* schema. Nothing compares it against the real one. A mismatch in column presence,
type, precision, nullability, key definition, delete rule, default or index produces no build error and no
startup error — it produces wrong data, or a failed insert, at the first write.

**Therefore: authoritative schema extraction is a GATE, not a nicety.** A `sys.columns` query against the
attached database — run on a supported Windows and LocalDB runtime, which is the only place the file can
be attached — must complete and be reconciled against the EF Core model **before** any Identity data is
migrated. This document records the gate; it does not discharge it, and it deliberately did not attach the
database to try, because attaching the tracked `.mdf` and `.ldf` files would cause SQL Server to write to
them and dirty the working tree, which [§1.3](#13-the-no-modification-constraint) forbids.

Two further facts make the gate wider than the Identity store:

- **The same extraction gates the catalogue schema**, for the same reason and with no probe evidence at
  all behind it.
- **The committed credential stores are simultaneously the only authoritative schema evidence and a
  Critical security finding** — [09 §8.2](09-security-assessment.md) states the collision directly:
  remediating F-09-34 by removing them destroys the evidence F-09-05's successor migration depends on,
  unless the two are sequenced. That sequencing is [03](03-modernization-roadmap.md)'s and belongs on its
  critical path.

**Successor: explicit migrations reconciled against an extracted schema.**

**Owner:** [05](05-aspnet-core-migration-approach.md) for the Identity migration design and the
reconciliation; [03](03-modernization-roadmap.md) for placing the extraction before the migration.

### F-12-22 — No usable schema baseline exists

**Editions:** MVC 5 (nothing ships) and MVC 4 (two copies of an unusable script).

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
`GetCartId(HttpContextBase context)` [:161] — or the MVC controller's own `HttpContext` and `Session`
properties, as at [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:17] and
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:39]. The session *mechanism* still changes —
[11 §3.1](11-cloud-readiness-assessment.md) owns that — but the *access pattern* ports.

### P-12-02 — The one genuinely unportable method signature is already dead code

`ShoppingCart.GetCart(MusicStoreEntities db, Controller controller)`
[src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:29] takes a `System.Web.Mvc.Controller` — a type with no
target equivalent, and the only place in the model layer that names one. It would be an awkward port.

It is also **provably unreferenced**. All **six** call sites use the `HttpContextBase` overload at [:21]:

```bash
git grep -n 'GetCart(store' -- 'src/MVC5/*.cs' | wc -l              # -> 6
```

— at [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:35],
[src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:17], [:41], [:58], [:89] and
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
[src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:21], called fire-and-forget from `ConfigureApp` [:18].
An `async void` method's exceptions cannot be observed by its caller, so a provisioning failure at startup
is silently lost — which is [09 §3.6](09-security-assessment.md)'s finding, not this document's. It is
recorded here only to bound the favourable claim: one method, one location, and it is a method whose
whole existence is being retired along with the credential it reads.

---

## 7. Roll-up and handoff

### 7.1 Distribution

| Group | Count | Discovered by | IDs |
| --- | --- | --- | --- |
| One — no successor or the API is gone | **14** | The compiler, for free | F-12-01 to F-12-14 |
| Two — the successor exists, its default differs | **8** | Only by someone who was told to look | F-12-15 to F-12-22 |
| Portability findings in the application's favour | 4 | — | P-12-01 to P-12-04 |

By edition: **17** blockers touch MVC 5, the migration source; **7** touch MVC 4 and **6** touch MVC 3,
counting the six that hold in all three. Group two is where the count understates the work — eight
blockers, but F-12-15 alone carries **nine distinct sites** across six files, and each needs an individual
decision.

### 7.2 The four statements that change what the strategy documents can assume

Called out separately because each one invalidates an assumption a downstream deliverable would otherwise
make:

1. **A namespace substitution table is not a migration plan.** Six of the fourteen compile-time blockers
   — F-12-02, F-12-03, F-12-04, F-12-05, F-12-10, F-12-11 — are constructs whose files are **deleted**
   rather than rewritten, so their `using` directives never get substituted at all. Two more, F-12-07 and
   F-12-09, are rewrites where the *signature* changes, not the namespace. Any plan built around
   old-namespace-to-new-namespace mapping silently omits eight of fourteen.
2. **The three constructs most likely to be misfiled as runtime problems are compile-time breaks.** The
   synchronous `TryUpdateModel` (F-12-07), the three `HttpNotFound()` calls (F-12-08) and `ChallengeResult`
   (F-12-09) all fail the **build**, because the overload, the method and the base type respectively do
   not exist in the target. They are not behavioural risks and must not be budgeted or tested as if they
   were. Section 2.3 and section 3 assign each of them explicitly, with the reason.
3. **Group two cannot be discharged by a policy statement.** "Use eager loading", "set the JSON naming
   policy" and "remove the `Dispose` overrides" are each true and each insufficient, because the work is
   per-site and the sites are not derivable from the policy. F-12-15's nine sites, F-12-16's five
   JavaScript reads and F-12-20's four overrides are the actionable units, and
   [05](05-aspnet-core-migration-approach.md) must resolve them individually.
4. **No schema baseline exists, and the only authoritative source is also a Critical security finding.**
   F-12-21 and F-12-22 together mean the data workstream **starts** with extraction, not with an EF Core
   initial migration — and [09 §8.2](09-security-assessment.md)'s collision between removing the committed
   credential stores and needing them as schema evidence has to be sequenced on
   [03](03-modernization-roadmap.md)'s critical path rather than discovered late.

### 7.3 What this document deliberately leaves open

Three decisions belong to [05](05-aspnet-core-migration-approach.md) and are recorded here as open rather
than resolved, so that nobody reads an enumeration as a design:

- **Whether the disabled external-login surface is mapped or removed** (F-12-09). Both are defensible; the
  registrations are inert either way.
- **Whether the JSON naming fix is scoped to the one response model or applied as an application-wide
  serializer policy** (F-12-16).
- **Whether the two `DbContext` classes remain separate** (F-12-19). This document records that neither
  convention survives, not what replaces the boundary.

---

## 8. Reproducibility appendix

Every command in this document, collected for re-execution. All are **read-only**: none writes to the
working tree, none contacts the network, and none attaches a database. Run from the repository root.
POSIX forms, executed on this Windows host through the bundled Git-for-Windows `bash`.

```bash
# --- F-12-01  MVC 3's SQL Server Compact database is not in the repository -----
git ls-files '*.sdf' | wc -l                                            # -> 0
git ls-files 'src/MVC3/MvcMusicStore-Assets/Data/*'                     # -> an .mdf, not an .sdf

# --- F-12-02  Bundling: five registrations, eleven view call sites -------------
git grep -n 'bundles.Add' -- 'src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs' | wc -l   # -> 5
git grep -n '@Scripts.Render' -- 'src/MVC5/*.cshtml' | wc -l            # -> 10
git grep -n '@Styles.Render'  -- 'src/MVC5/*.cshtml' | wc -l            # -> 1

# --- F-12-04  The Global.asax markup declaration, one per edition --------------
git ls-files '*Global.asax'                                             # -> 3 paths

# --- F-12-08  The HttpNotFound census (three, not "several") -------------------
git grep -n 'HttpNotFound()' -- 'src/MVC5/*.cs' | wc -l                 # -> 3

# --- F-12-09  Every external provider registration is commented out -----------
git grep -nE '^\s*//app\.Use' -- 'src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs' | wc -l  # -> 4

# --- F-12-12  Assembly metadata as source, one per edition --------------------
git ls-files '*AssemblyInfo.cs'                                         # -> 3 paths

# --- F-12-15  Which navigations are lazy-loadable, and which are not ----------
git grep -nE 'virtual|List<Album>' -- 'src/MVC5/MvcMusicStore/Models/Album.cs' \
    'src/MVC5/MvcMusicStore/Models/Cart.cs' 'src/MVC5/MvcMusicStore/Models/Genre.cs'
# -> Album.cs:30,:31,:32 and Cart.cs:17 are virtual; Genre.cs:10 is NOT

# --- F-12-15  The dereference census: every navigation read in source and views
git grep -nE '\.(Album|Genre|Artist)\.' -- 'src/MVC5/MvcMusicStore/Controllers' \
    'src/MVC5/MvcMusicStore/Models/ShoppingCart.cs' 'src/MVC5/MvcMusicStore/Views'
# Read the result against the eager-loading sites, which are the two exclusions:
git grep -n 'Include(' -- 'src/MVC5/MvcMusicStore/Controllers'
# -> StoreController.cs:30 (string form, still valid in EF Core) and
#    StoreManagerController.cs:22 (typed form) -- their views are NOT affected

# --- F-12-16  Newtonsoft.Json is pinned but never called from source ----------
git grep -nE 'Newtonsoft|JsonConvert|JsonProperty|JsonSerializer' -- '*.cs' '*.cshtml' | wc -l   # -> 0
git grep -n 'Newtonsoft' -- 'src/MVC5/MvcMusicStore/packages.config'    # -> :27, version 5.0.6

# --- F-12-17  Path casing: three files, one capital S ------------------------
git ls-files 'src/MVC5/MvcMusicStore/Content/*'
# -> Site.css, bootstrap.css, bootstrap.min.css   (the bundle registers "site.css")

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

# --- The constraint this work was held to ---------------------------------
git status --porcelain      # -> nothing but new files under docs/modernization/
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
| F-12-09 `ChallengeResult` | 05 (map or remove — the choice is 05's), 09 §6.11 (disabled surface) |
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
| P-12-01 to P-12-04 Favourable findings | 07 (scope and effort), 05 (context, query and async transitions) |

---

*Deliverable 12 of 13. Consumes deliverables [09](09-security-assessment.md),
[10](10-build-and-deployment-requirements.md) and [11](11-cloud-readiness-assessment.md); feeds the three
strategy documents [04](04-dotnet8-migration-strategy.md), [05](05-aspnet-core-migration-approach.md) and
[06](06-azure-hosting-recommendations.md). Index and requirement map: [README](README.md). No user rules
were provided for this project. Every claim above carries an inline file location, every count is
reproducible by the command stated beside it, and no repository file outside `docs/modernization/` was
created, modified or deleted in its production.*
