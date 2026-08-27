# 02 — Dependency Inventory

**Deliverable 02 of 13** · MvcMusicStore modernization assessment · *Assessment only — no repository file is modified by this document.*

Answers two of the user's fourteen requirements: **"Framework and package dependencies"** (analyze) and **"Dependency inventory"** (produce). Together with [01 — Architecture Overview](01-architecture-overview.md) this is a *foundation* document: the other eleven deliverables cite it rather than re-deriving its numbers.

---

## 1. Purpose, scope and authoring contract

### 1.1 What this document is

A complete, verbatim catalogue of every declared dependency of all three shipped editions of MvcMusicStore, as the repository declares them **today**. It covers three classes of dependency, because the NuGet manifests alone do not describe what the applications need:

1. **NuGet package pins** — the 63 `<package>` entries across the three `packages.config` files (§3).
2. **Dependencies NuGet does not resolve** — .NET Framework assemblies, machine-installed products and native providers referenced without any package backing them (§4).
3. **Build-tool and restore-configuration dependencies** — the committed NuGet client and the restore wiring that consumes it (§5, §6).

It then records what the repository does *not* say: no lockfile, no configured package source, no dependency-scanning configuration (§6, §7, §8).

### 1.2 What this document deliberately does not contain

Under the single-owner rule (AAP 0.11.4) each cross-cutting decision is stated in full by exactly one deliverable and cross-referenced by the rest. This document owns the **current** inventory and nothing else. In particular it contains **no target-state package version, no successor version, no target framework and no SDK band** — not once, anywhere. Those belong to:

| Question | Owner |
| --- | --- |
| Target framework, SDK band, per-package migration outcome, target-state `NuGet.config`, target-state lockfile policy | [04 — .NET 8 Migration Strategy](04-dotnet8-migration-strategy.md) |
| Client-library acquisition mechanism and the Bootstrap markup work implied by upgrading it | [05 — ASP.NET Core Migration Approach](05-aspnet-core-migration-approach.md) |
| Hosting target and deployment model | [06 — Azure Hosting Recommendations](06-azure-hosting-recommendations.md) |
| Effort bands and the risk register | [07 — Effort, Risks and Sequencing](07-effort-risks-sequencing.md) |
| Debt framing, severity and ownership of the items below | [08 — Technical Debt Register](08-technical-debt-register.md) |
| Per-edition build outcome and restore posture | [10 — Build and Deployment Requirements](10-build-and-deployment-requirements.md) |
| Which of these packages has no successor, and the serialization consequence | [12 — Migration Blockers](12-migration-blockers.md) |

Where a fact below has a forward consequence, this document names the owning deliverable and stops there.

### 1.3 The no-modification constraint

The user directed *"Do not make code changes initially"*, and the project's attached environment setup instructions independently restate the same gate (*"Do not modify code until assessment and modernization plan are approved"*). Accordingly, and per AAP 0.5.4: **no `packages.config` was edited, no package was added, upgraded, downgraded or removed, no `packages/` directory was restored into the tree, and no project file gained or lost a reference.** The repository's dependency graph is byte-identical before and after this document was written.

Every figure below was obtained by reading files and by read-only `git` queries, with one flagged exception: the advisory evidence in §8.2 required a `nuget restore`, which downloads packages into the gitignored `packages/` directories. That restore was performed, its output recorded, and the generated `packages/`, `bin/` and `obj/` directories removed afterwards, leaving `git status --porcelain` reporting nothing but the new file under `docs/modernization/`. No tracked file was read-modified at any point.

This is a catalogue, not a curation.

### 1.4 Authoring contract, and the absence of user rules

`review_rules` returns exactly **"No user rules provided."** There is therefore no project rule to name, summarize or comply with, and no file forced into scope by one. The absence is not licence to lower the bar; this document is held to enterprise-standard assessment practice and to the following contracts, which stand in place of rules:

- **Repository evidence is primary.** Every as-is claim carries an inline `[<path>:<locator>]` citation placed at the claim, never collected in a trailing reference list. Paths are repository-relative and resolve in the checkout.
- **A repository-wide claim carries its reproducing command** next to the claim, because a count or an absence has no single line to point at. That is the stronger form of evidence: a reader can re-run it.
- **Exact versions only.** Every version string below is transcribed character-for-character from the manifest that declares it. No ranges, no rounding, no "or later". `1.0.0.0` is not `1.0.0`, and `1.0` is not `1.0.0`.
- **The Technical Specification is secondary.** Sections of it may be cited *alongside* repository evidence, never instead of it. §6.2 below records a place where the specification and the repository disagree, and resolves it in the repository's favour.

**Reproducing the commands on this host.** The canonical commands quoted throughout are the POSIX forms fixed by AAP 0.11.3. The verification host for this assessment is Windows; they were executed through the Git-for-Windows `bash` bundled on the host, from the repository root, and the outputs quoted are the outputs observed there.

---

## 2. Inventory at a glance

| Measure | Value | Evidence |
| --- | --- | --- |
| NuGet package pins, all editions | **63** | `git ls-files '*packages.config' \| grep -v '/packages/' \| xargs grep -h '<package ' \| wc -l` → `63` |
| — MVC 5 | 28 | [src/MVC5/MvcMusicStore/packages.config:3-30] |
| — MVC 4 | 29 | [src/MVC4/MvcMusicStore/packages.config:3-31] |
| — MVC 3 | 6 | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config:3-8] |
| Distinct registries | 1 (`nuget.org`) | every pin is a public identifier; see §2.1 |
| Version ranges or floating versions | 0 | every `version=` attribute is a single exact release |
| Lockfiles | **0** | `git ls-files 'packages.lock.json' \| wc -l` → `0` (§7) |
| Configured package sources | **0** | no `packageSources` element anywhere (§6) |
| Committed NuGet clients | 1 | [src/MVC4/MvcMusicStore/.nuget/NuGet.exe], 630,784 bytes (§5) |
| Tracked files under committed `packages/` trees | **215** | `git ls-files \| grep -c '/packages/'` → `215` (§7.2) |
| Dependency-scanning configuration | **0** | no `.github/`, no Dependabot or Renovate config, no analyzer package (§8.2) |
| Pins named by NuGet's restore audit | **14 of 63** | 43 `NU1902`/`NU1903` warnings on 2026-08-27 with NuGet 6.11.1.2 (§8.2) |

### 2.1 Registry and provenance statement

**All 63 pins are public package identifiers verifiable on nuget.org.** Every one declares a single exact version with no range, no wildcard and no floating component. **Nothing in the repository indicates an internal, private or vendored-feed package**: every identifier is a well-known public one, and no manifest, project file or configuration file names a private feed.

Two qualifications belong with that statement rather than after it, and both are consequential:

- The claim above is about the **identifiers**, which are public. It is *not* a claim about which feed a restore actually contacts — the repository configures no source at all, so the effective source set is unknowable from the repository. §6 states that finding in full.
- The absence of a lockfile means the 63 exact pins constrain only the **direct** graph. Transitive resolution is not pinned and not reproducible from the repository. §7 states that finding in full.

### 2.2 How the three manifests differ in kind, not just in content

| | MVC 5 | MVC 4 | MVC 3 |
| --- | --- | --- | --- |
| Pins | 28 | 29 | 6 |
| `targetFramework` attribute | on all 28, value `net45` | on all 29, value `net45` | **absent from all 6** |
| Project's target framework | `v4.8` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:16] | `v4.5` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:16] | `v4.0` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:15] |
| Manifest and project agree? | **No** — manifest says `net45`, project says `v4.8` | Yes | No attribute to agree or disagree |
| Pins whose payload the project references by `HintPath` | 22 of 28 | 20 of 29 | 1 of 6 |
| `packages/` payload committed to source control | none | 169 files, 29 folders | 46 files, 6 folders |
| MSBuild-integrated restore wired | import present but **conditional** [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:295] | **unconditional** import [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:360] with `<RestorePackages>true</RestorePackages>` [:24] | not wired |
| `.nuget` folder present | **no** — although the solution declares one (§5.2) | yes [src/MVC4/MvcMusicStore/.nuget/] | no |

The three editions are therefore not three instances of one dependency-management approach; they are three different approaches, and a single statement about "how MvcMusicStore restores packages" would be wrong for at least one of them. Deliverable 10 owns the build consequence of that.

---

## 3. NuGet package pins

Each table lists every `<package>` entry of one manifest, **in the order the file declares them**, so that a reviewer can diff the table against the file line by line. The `Manifest line` column is the per-row citation: read it as `[<the manifest named in the heading>:<line>]`. Registry is `nuget.org` for every row, per §2.1.

`Purpose` states what the package delivers to *this* repository — which assemblies the project actually references from it, or that it delivers content rather than an assembly. It is not a general description of the package.

### 3.1 MVC 5 — 28 pins

Manifest: **[src/MVC5/MvcMusicStore/packages.config]**, 31 lines, pins at `:3-30`.

| Registry | Package | Version | Purpose | Manifest line |
| --- | --- | --- | --- | --- |
| nuget.org | Antlr | `3.4.1.9004` | Parser runtime; the project references `Antlr3.Runtime` from it [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:108-111]. Present as the minification stack's dependency, not called from application code | `:3` |
| nuget.org | bootstrap | `3.0.0` | CSS and JS UI framework; content only — delivers `Content/bootstrap.css`, `Scripts/bootstrap.js` and the Glyphicons font set under `fonts/` | `:4` |
| nuget.org | EntityFramework | `6.0.0` | ORM. Supplies **two** referenced assemblies: `EntityFramework.dll` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:114-116] and `EntityFramework.SqlServer.dll` [:117-119] | `:5` |
| nuget.org | jQuery | `1.10.2` | Client JS library; content only — delivers `Scripts/jquery-1.10.2.js` and its minified and IntelliSense companions | `:6` |
| nuget.org | jQuery.Validation | `1.11.1` | Client-side validation plugin; content only — `Scripts/jquery.validate.js` | `:7` |
| nuget.org | Microsoft.AspNet.Identity.Core | `1.0.0` | ASP.NET Identity 1.0 core abstractions and `UserManager` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:120-122] | `:8` |
| nuget.org | Microsoft.AspNet.Identity.EntityFramework | `1.0.0` | EF-backed Identity 1.0 user store [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:126-128]; defines the schema of the credential store the application ships | `:9` |
| nuget.org | Microsoft.AspNet.Identity.Owin | `1.0.0` | Bridges Identity 1.0 onto the OWIN authentication pipeline [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:123-125] | `:10` |
| nuget.org | Microsoft.AspNet.Mvc | `5.0.0` | The MVC framework; supplies `System.Web.Mvc` 5.0.0.0 [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:76-79] | `:11` |
| nuget.org | Microsoft.AspNet.Razor | `3.0.0` | Razor 3 view engine; supplies `System.Web.Razor` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:83-86] | `:12` |
| nuget.org | Microsoft.AspNet.Web.Optimization | `1.1.1` | Bundling and minification; supplies `System.Web.Optimization` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:80-82] | `:13` |
| nuget.org | Microsoft.AspNet.WebPages | `3.0.0` | Web Pages runtime. Supplies **four** referenced assemblies: `System.Web.Helpers` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:72-75], `System.Web.WebPages` [:87-90], `System.Web.WebPages.Deployment` [:91-94] and `System.Web.WebPages.Razor` [:95-98] | `:14` |
| nuget.org | Microsoft.jQuery.Unobtrusive.Validation | `3.0.0` | Unobtrusive validation adapters; content only — `Scripts/jquery.validate.unobtrusive.js` | `:15` |
| nuget.org | Microsoft.Owin | `2.0.0` | Katana OWIN host abstractions [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:132-134] | `:16` |
| nuget.org | Microsoft.Owin.Host.SystemWeb | `2.0.0` | Hosts the OWIN pipeline on the `System.Web` request pipeline [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:135-137] | `:17` |
| nuget.org | Microsoft.Owin.Security | `2.0.0` | Base types shared by all authentication middleware [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:138-140] | `:18` |
| nuget.org | Microsoft.Owin.Security.Cookies | `2.0.0` | Cookie authentication — **the only authentication middleware the application actually enables**, at [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:14-18] and [:20] | `:19` |
| nuget.org | Microsoft.Owin.Security.Facebook | `2.0.0` | External sign-in provider; its registration is commented out [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:31-33] — dormant (§3.1.2) | `:20` |
| nuget.org | Microsoft.Owin.Security.Google | `2.0.0` | External sign-in provider; registration commented out [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:35] — dormant | `:21` |
| nuget.org | Microsoft.Owin.Security.MicrosoftAccount | `2.0.0` | External sign-in provider; registration commented out [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:23-25] — dormant | `:22` |
| nuget.org | Microsoft.Owin.Security.OAuth | `2.0.0` | OAuth authorization-server and bearer-token **infrastructure**. Not a social provider and not the package any of the four commented registrations would use (§3.1.2) | `:23` |
| nuget.org | Microsoft.Owin.Security.Twitter | `2.0.0` | External sign-in provider; registration commented out [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:27-29] — dormant | `:24` |
| nuget.org | Microsoft.Web.Infrastructure | `1.0.0.0` | Dynamic HTTP-module registration at runtime [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:64-67]. Note the four-part version — the manifest says `1.0.0.0`, not `1.0.0` | `:25` |
| nuget.org | Modernizr | `2.6.2` | Browser feature detection; content only — `Scripts/modernizr-2.6.2.js` | `:26` |
| nuget.org | Newtonsoft.Json | `5.0.6` | JSON serializer. Referenced by the project [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:99-102] but **never called from application source** (§3.1.3) | `:27` |
| nuget.org | Owin | `1.0` | The `IAppBuilder` abstraction (`Owin.dll`) [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:129-131]. The manifest says `1.0` — two parts, not `1.0.0` | `:28` |
| nuget.org | Respond | `1.2.0` | Media-query polyfill for Internet Explorer 8; content only — `Scripts/respond.js` | `:29` |
| nuget.org | WebGrease | `1.5.2` | Minification engine behind `System.Web.Optimization` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:104-107] | `:30` |

Verify the count and the exact strings:

```bash
grep -c '<package ' src/MVC5/MvcMusicStore/packages.config          # -> 28
grep -o 'id="[^"]*" version="[^"]*"' src/MVC5/MvcMusicStore/packages.config
```

#### 3.1.1 Finding — the manifest and the project disagree about the platform

**Every one of the 28 entries declares `targetFramework="net45"`** [src/MVC5/MvcMusicStore/packages.config:3-30] while the project declares `<TargetFrameworkVersion>v4.8</TargetFrameworkVersion>` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:16].

```bash
grep -c 'targetFramework="net45"' src/MVC5/MvcMusicStore/packages.config   # -> 28, i.e. all of them
```

This is a finding, not a footnote. The `targetFramework` attribute records the framework each package was *installed against*, and it is what a `packages.config`-era restore and reinstall use to select the correct lib folder from a package payload. A manifest that believes the project is `net45` while the project builds as `v4.8` means the recorded install context no longer describes the project — so which asset group a reinstall selects is determined by a stale value rather than by the project's real target. Every `HintPath` in the project points into a `lib\net45\` folder, consistent with the manifest and not with the project. MVC 5 is the sole migration source (AAP 0.3.1), which is why this particular disagreement matters more than the same class of drift would elsewhere.

A second, independent framework-version disagreement exists inside MVC 5's own `Web.config`. It is not a dependency fact and it is **not** restated here: [12 — Migration Blockers](12-migration-blockers.md) owns it.

#### 3.1.2 Finding — five provider-family packages ship, four external logins are dormant, and one of the five is not a provider at all

The distinction here is easy to get wrong, so it is stated precisely.

MVC 5's authentication configuration enables exactly two things: cookie authentication [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:14-18] and the external sign-in cookie [:20]. Four external-provider registrations sit immediately below, all commented out [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:22-35] — a fourteen-line block:

| Commented registration | Location | Package that would serve it | Pin |
| --- | --- | --- | --- |
| `app.UseMicrosoftAccountAuthentication(...)` | [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:23-25] | Microsoft.Owin.Security.MicrosoftAccount | `2.0.0` |
| `app.UseTwitterAuthentication(...)` | [:27-29] | Microsoft.Owin.Security.Twitter | `2.0.0` |
| `app.UseFacebookAuthentication(...)` | [:31-33] | Microsoft.Owin.Security.Facebook | `2.0.0` |
| `app.UseGoogleAuthentication()` | [:35] | Microsoft.Owin.Security.Google | `2.0.0` |

So **four** dormant provider packages are pinned, referenced by the project and deployed with it, serving nothing.

The **fifth** `Microsoft.Owin.Security.*` package, `Microsoft.Owin.Security.OAuth` `2.0.0` [src/MVC5/MvcMusicStore/packages.config:23], is **not** a social provider and is not the package any of those four registrations would use. It supplies OAuth authorization-server and bearer-token infrastructure. Counting it as a fifth dormant social provider — or, in the other direction, treating the four provider packages as "the OAuth package" — is a common and material error: the two have different removal consequences, and [04 — .NET 8 Migration Strategy](04-dotnet8-migration-strategy.md) assigns each package its own outcome on that basis.

The same pattern, with a different stack, appears in MVC 4 (§3.2.3).

#### 3.1.3 Finding — Newtonsoft.Json 5.0.6 ships but is never called from application source

`Newtonsoft.Json` `5.0.6` is pinned [src/MVC5/MvcMusicStore/packages.config:27] and referenced by the project [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:99-102], so it is restored, copied to `bin` and deployed. **No application source file references it.** A repository-wide search across every tracked C# and Razor file, excluding the committed package payloads, returns nothing:

```bash
git ls-files '*.cs' '*.cshtml' | grep -v '/packages/' | xargs grep -lE 'Newtonsoft|JsonConvert' | wc -l   # -> 0
```

The application's one JSON-producing endpoint returns an MVC `JsonResult`, and in MVC 5 that result type serializes through `JavaScriptSerializer` rather than through Newtonsoft.Json. The package is therefore **template baggage** — it arrived with the project template and nothing consumes it.

Two consequences must not be conflated, and this document deliberately draws only the first:

- **As an inventory fact:** the pin exists, is deployed, and has no consumer in application code.
- **As a migration fact:** because nothing calls it, removing it is not a serializer change — but the endpoint's serialized output *does* change for an unrelated reason. That consequence, and the JSON property-naming behaviour behind it, is owned by [12 — Migration Blockers](12-migration-blockers.md) and is not analyzed here.

#### 3.1.4 Six of MVC 5's 28 pins deliver content rather than an assembly

Matching each pin's exact package folder against the project's `HintPath` entries: 22 of the 28 pins have their payload referenced as an assembly; the remaining six deliver static files only — **bootstrap, jQuery, jQuery.Validation, Microsoft.jQuery.Unobtrusive.Validation, Modernizr and Respond**.

Their pinned versions are corroborated a second way, by the vendored filenames under the application root: `Scripts/jquery-1.10.2.js` matches the jQuery pin `1.10.2`, `Scripts/modernizr-2.6.2.js` matches Modernizr `2.6.2`, and `Content/bootstrap.css` with `fonts/glyphicons-halflings-regular.*` matches bootstrap `3.0.0`'s payload shape. The version a browser is served is the version the manifest pins.

```bash
git ls-files 'src/MVC5/MvcMusicStore/Scripts/*' 'src/MVC5/MvcMusicStore/Content/*' 'src/MVC5/MvcMusicStore/fonts/*'
```

The distinction matters to the inventory because a content-only package has no assembly to bind, no binding redirect and no runtime version to reconcile — its entire footprint is files already committed to the tree. The acquisition mechanism these six would need after a migration is owned by [05 — ASP.NET Core Migration Approach](05-aspnet-core-migration-approach.md).

### 3.2 MVC 4 — 29 pins

Manifest: **[src/MVC4/MvcMusicStore/packages.config]**, 32 lines, pins at `:3-31`. Every entry declares `targetFramework="net45"`, matching the project's `<TargetFrameworkVersion>v4.5</TargetFrameworkVersion>` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:16]; unlike MVC 5, this manifest and its project agree.

```bash
grep -c 'targetFramework="net45"' src/MVC4/MvcMusicStore/packages.config    # -> 29, i.e. all of them
```

| Registry | Package | Version | Purpose | Manifest line |
| --- | --- | --- | --- | --- |
| nuget.org | DotNetOpenAuth.AspNet | `4.0.3.12153` | ASP.NET integration for the DotNetOpenAuth stack [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:127-130]; serves the commented-out external logins (§3.2.3) | `:3` |
| nuget.org | DotNetOpenAuth.Core | `4.0.3.12153` | Core protocol types shared by the OAuth and OpenID assemblies [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:131-134] | `:4` |
| nuget.org | DotNetOpenAuth.OAuth.Consumer | `4.0.3.12153` | OAuth consumer (relying-party) implementation [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:135-138] | `:5` |
| nuget.org | DotNetOpenAuth.OAuth.Core | `4.0.3.12153` | OAuth core; supplies `DotNetOpenAuth.OAuth.dll` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:139-142] — note the assembly name differs from the package id | `:6` |
| nuget.org | DotNetOpenAuth.OpenId.Core | `4.0.3.12153` | OpenID core; supplies `DotNetOpenAuth.OpenId.dll` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:143-146] | `:7` |
| nuget.org | DotNetOpenAuth.OpenId.RelyingParty | `4.0.3.12153` | OpenID relying-party implementation [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:147-150] | `:8` |
| nuget.org | EntityFramework | `5.0.0` | ORM; supplies `EntityFramework.dll` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:65-67]. Declared again in configuration as `EntityFramework, Version=5.0.0.0` [src/MVC4/MvcMusicStore/Web.config:9] | `:9` |
| nuget.org | jQuery | `1.7.1.1` | Client JS library; content only — `Scripts/jquery-1.7.1.js` | `:10` |
| nuget.org | jQuery.UI.Combined | `1.8.20.1` | jQuery UI widget library; content only. **Source of MVC 4's nested theme tree** (§3.2.4) | `:11` |
| nuget.org | jQuery.Validation | `1.9.0.1` | Client-side validation plugin; content only — `Scripts/jquery.validate.js` | `:12` |
| nuget.org | knockoutjs | `2.1.0` | Client-side MVVM library; content only — `Scripts/knockout-2.1.0.js` | `:13` |
| nuget.org | Microsoft.AspNet.Mvc | `4.0.20710.0` | The MVC framework; supplies `System.Web.Mvc` 4.0.0.0 [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:92-95] | `:14` |
| nuget.org | Microsoft.AspNet.Razor | `2.0.20710.0` | Razor 2 view engine; supplies `System.Web.Razor` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:99-102] | `:15` |
| nuget.org | Microsoft.AspNet.Web.Optimization | `1.0.0` | Bundling and minification; supplies `System.Web.Optimization` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:96-98] | `:16` |
| nuget.org | Microsoft.AspNet.WebApi | `4.0.20710.0` | **Metapackage** — its committed payload is the `.nupkg` alone, with no `lib` folder, so it contributes no assembly of its own and exists to pull in the three packages below (§3.2.2) | `:17` |
| nuget.org | Microsoft.AspNet.WebApi.Client | `4.0.20710.0` | Web API client libraries; supplies `System.Net.Http.Formatting` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:77-79] | `:18` |
| nuget.org | Microsoft.AspNet.WebApi.Core | `4.0.20710.0` | Web API runtime; supplies `System.Web.Http` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:86-88] | `:19` |
| nuget.org | Microsoft.AspNet.WebApi.WebHost | `4.0.20710.0` | Hosts Web API on `System.Web`; supplies `System.Web.Http.WebHost` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:89-91] | `:20` |
| nuget.org | Microsoft.AspNet.WebPages | `2.0.20710.0` | Web Pages runtime. Supplies **four** referenced assemblies: `System.Web.Helpers` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:82-85], `System.Web.WebPages` [:103-106], `System.Web.WebPages.Deployment` [:107-110] and `System.Web.WebPages.Razor` [:111-114] | `:21` |
| nuget.org | Microsoft.AspNet.WebPages.Data | `2.0.20710.0` | Web Pages data helpers; supplies `WebMatrix.Data` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:115-118] | `:22` |
| nuget.org | Microsoft.AspNet.WebPages.OAuth | `2.0.20710.0` | `OAuthWebSecurity` surface; supplies `Microsoft.Web.WebPages.OAuth` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:119-122] — the type the commented registrations of §3.2.3 call | `:23` |
| nuget.org | Microsoft.AspNet.WebPages.WebData | `2.0.20710.0` | SimpleMembership provider; supplies `WebMatrix.WebData` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:123-126]. This is MVC 4's membership stack | `:24` |
| nuget.org | Microsoft.jQuery.Unobtrusive.Ajax | `2.0.20710.0` | Unobtrusive AJAX adapters; content only — `Scripts/jquery.unobtrusive-ajax.js` | `:25` |
| nuget.org | Microsoft.jQuery.Unobtrusive.Validation | `2.0.20710.0` | Unobtrusive validation adapters; content only — `Scripts/jquery.validate.unobtrusive.js` | `:26` |
| nuget.org | Microsoft.Net.Http | `2.0.20710.0` | `HttpClient` backport for .NET 4.0. Its committed payload carries `lib/net40` assemblies and a `lib/net45/_._` placeholder, so **on this `net45` project it contributes nothing** (§3.2.2) | `:27` |
| nuget.org | Microsoft.Web.Infrastructure | `1.0.0.0` | Dynamic HTTP-module registration [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:68-71]. Four-part version, as in MVC 5 | `:28` |
| nuget.org | Modernizr | `2.5.3` | Browser feature detection; content only — `Scripts/modernizr-2.5.3.js` | `:29` |
| nuget.org | Newtonsoft.Json | `4.5.6` | JSON serializer [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:72-74]; a Web API formatter dependency. Not called from MVC 4 application source either — the same command as §3.1.3 returns zero across the whole repository | `:30` |
| nuget.org | WebGrease | `1.1.0` | Minification engine behind `System.Web.Optimization`. Supplies **two** referenced assemblies here: `WebGrease.dll` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:151-154] and `Antlr3.Runtime.dll` [:155-158] (§3.2.5) | `:31` |

```bash
grep -c '<package ' src/MVC4/MvcMusicStore/packages.config          # -> 29
grep -o 'id="[^"]*" version="[^"]*"' src/MVC4/MvcMusicStore/packages.config
```

#### 3.2.1 A shared release stamp is not a single version

Eleven of MVC 4's pins carry the version `4.0.20710.0` or `2.0.20710.0` — the same `20710` release stamp from the ASP.NET MVC 4 / Web API wave. It is worth being explicit that **this is a shared build stamp, not one version**: the MVC and Web API packages are at `4.0.20710.0` while the Razor, WebPages and unobtrusive-script packages from the same wave are at `2.0.20710.0`, and the remaining eighteen pins share neither. Reading `20710` as a single "framework version" for MVC 4 would produce two wrong version strings out of eleven.

The manifest's own numbers, grouped:

| Version string | Pins | Packages |
| --- | --- | --- |
| `4.0.20710.0` | 5 | Microsoft.AspNet.Mvc; Microsoft.AspNet.WebApi; Microsoft.AspNet.WebApi.Client; Microsoft.AspNet.WebApi.Core; Microsoft.AspNet.WebApi.WebHost |
| `2.0.20710.0` | 8 | Microsoft.AspNet.Razor; Microsoft.AspNet.WebPages; Microsoft.AspNet.WebPages.Data; Microsoft.AspNet.WebPages.OAuth; Microsoft.AspNet.WebPages.WebData; Microsoft.jQuery.Unobtrusive.Ajax; Microsoft.jQuery.Unobtrusive.Validation; Microsoft.Net.Http |
| `4.0.3.12153` | 6 | the six DotNetOpenAuth packages |
| ten further distinct versions | 10 | EntityFramework `5.0.0`; jQuery `1.7.1.1`; jQuery.UI.Combined `1.8.20.1`; jQuery.Validation `1.9.0.1`; knockoutjs `2.1.0`; Microsoft.AspNet.Web.Optimization `1.0.0`; Microsoft.Web.Infrastructure `1.0.0.0`; Modernizr `2.5.3`; Newtonsoft.Json `4.5.6`; WebGrease `1.1.0` |

Thirteen pins share the `20710` stamp across **two** different version strings, and sixteen do not carry it at all; 5 + 8 + 6 + 10 = 29. This grouping exists only to prevent the stamp being mistaken for a version — the authoritative per-pin values are the table in §3.2.

#### 3.2.2 Finding — four Web API packages support a route with zero implementations, and one of the four contributes nothing

MVC 4 pins four Web API packages [src/MVC4/MvcMusicStore/packages.config:17-20] and registers a Web API route: `config.Routes.MapHttpRoute` with `name: "DefaultApi"` and `routeTemplate: "api/{controller}/{id}"` [src/MVC4/MvcMusicStore/App_Start/WebApiConfig.cs:12-16].

**There is no `ApiController` implementation anywhere in the repository** — not in MVC 4, not in any edition:

```bash
git ls-files '*.cs' | grep -v '/packages/' | xargs grep -n 'ApiController' | wc -l   # -> 0
```

The route is mapped and can never dispatch. Four packages, their assemblies and their transitive payload are restored, referenced and deployed to serve it.

Two of the four deserve their own note, because both are cases where the pin count overstates the delivered surface:

- **`Microsoft.AspNet.WebApi` `4.0.20710.0` is a metapackage.** Its committed payload consists of exactly one file, its own `.nupkg`, with no `lib` folder at all — verifiable directly, because MVC 4's `packages/` tree is committed: `git ls-files 'src/MVC4/MvcMusicStore/packages/Microsoft.AspNet.WebApi.4.0.20710.0/*'` returns only `Microsoft.AspNet.WebApi.4.0.20710.0.nupkg`. It contributes no assembly of its own; it exists to bring in the other three.
- **`Microsoft.Net.Http` `2.0.20710.0` contributes nothing to this project.** It is the `HttpClient` backport for .NET 4.0. The project references `System.Net.Http` and `System.Net.Http.WebRequest` **with no `HintPath`** [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:75-76] and [:80-81], so those resolve from the framework, not from the package. The package's own payload confirms the intent: `lib/net40/System.Net.Http.dll` exists, and the `net45` group is the single placeholder file `lib/net45/_._`, which is how a package declares "on this framework I supply nothing". Since the project targets `v4.5`, the pin is inert.

The debt framing and severity of the dead Web API scaffolding are owned by [08 — Technical Debt Register](08-technical-debt-register.md); the inventory fact is stated here.

#### 3.2.3 Finding — six DotNetOpenAuth packages support external logins that are all commented out

All six DotNetOpenAuth pins sit at `4.0.3.12153` [src/MVC4/MvcMusicStore/packages.config:3-8] and all six assemblies are referenced by the project [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:127-150]. What they exist to serve is entirely disabled:

| Commented registration | Location |
| --- | --- |
| `OAuthWebSecurity.RegisterMicrosoftClient(...)` | [src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs:17-19] |
| `OAuthWebSecurity.RegisterTwitterClient(...)` | [:21-23] |
| `OAuthWebSecurity.RegisterFacebookClient(...)` | [:25-27] |
| `OAuthWebSecurity.RegisterGoogleClient()` | [:29] |

`AuthConfig.RegisterAuth()` [src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs:12-30] therefore has an empty body once the comments are discounted. Six packages plus the `Microsoft.AspNet.WebPages.OAuth` pin that provides `OAuthWebSecurity` itself are deployed for a capability no request can reach.

This is the same shape as MVC 5's dormant-provider finding (§3.1.2) but a different stack: MVC 5 carries four dormant Katana provider packages, MVC 4 carries six DotNetOpenAuth packages plus the WebPages OAuth surface. The two are not interchangeable and neither substitutes for the other in a per-package outcome assessment.

#### 3.2.4 `jQuery.UI.Combined` `1.8.20.1` is the origin of MVC 4's nested asset tree

MVC 4's `Content` directory holds 55 tracked files, of which **54 sit under `Content/themes/`** — the jQuery UI base theme, delivered by this pin:

```bash
git ls-files 'src/MVC4/MvcMusicStore/Content/*' | wc -l                        # -> 55
git ls-files 'src/MVC4/MvcMusicStore/Content/themes/*' | wc -l                 # -> 54
git ls-files 'src/MVC4/MvcMusicStore/packages/jQuery.UI.Combined.1.8.20.1/*'   # its theme payload sits at Content/Content/themes/base/**
```

One content-only pin therefore accounts for the large majority of MVC 4's static-asset count, and for the fact that any pattern over MVC 4's `Content` directory must be recursive. The asset-migration sizing that depends on it is owned by [05 — ASP.NET Core Migration Approach](05-aspnet-core-migration-approach.md).

#### 3.2.5 MVC 4 has no Antlr pin — it takes `Antlr3.Runtime` out of the WebGrease payload

MVC 5 pins `Antlr` `3.4.1.9004` explicitly [src/MVC5/MvcMusicStore/packages.config:3] and references it from that package's own folder [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:108-111].

MVC 4 has **no `Antlr` pin at all**, yet still references `Antlr3.Runtime` — resolving it from inside the WebGrease package: `<HintPath>..\packages\WebGrease.1.1.0\lib\Antlr3.Runtime.dll</HintPath>` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:155-158].

The dependency is real and identical in kind across the two editions, but it is *declared* in one and *undeclared* in the other. An inventory built by reading `packages.config` alone would report MVC 4 as having no Antlr dependency, which is wrong. This is the smallest instance of the general problem §4 addresses.

### 3.3 MVC 3 — 6 pins

Manifest: **[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config]**, 9 lines, pins at `:3-8`.

**No entry carries a `targetFramework` attribute** — the attribute is absent from all six, not merely blank. This is the pre-NuGet-2.0 manifest form, consistent with the project's `<TargetFrameworkVersion>v4.0</TargetFrameworkVersion>` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:15] and with the 2011 vintage of the edition:

```bash
grep -c '<package ' src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config           # -> 6
grep -c 'targetFramework' src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config     # -> 0
```

With no recorded install framework, a reinstall has no manifest-supplied basis for choosing an asset group — it must infer one from the project. The consequence is a restore-posture concern rather than a dependency-graph one, and it belongs to [10 — Build and Deployment Requirements](10-build-and-deployment-requirements.md).

| Registry | Package | Version | Purpose | Manifest line |
| --- | --- | --- | --- | --- |
| nuget.org | jQuery | `1.5.1` | Client JS library; content only — `Scripts/jquery-1.5.1.js` | `:3` |
| nuget.org | jQuery.vsdoc | `1.5.1` | IntelliSense companion for jQuery `1.5.1`; content only, design-time only — `Scripts/jquery-1.5.1-vsdoc.js`. It has no runtime role at all | `:4` |
| nuget.org | jQuery.Validation | `1.8.0` | Client-side validation plugin; content only — `Scripts/jquery.validate.js` | `:5` |
| nuget.org | jQuery.UI.Combined | `1.8.11` | jQuery UI widget library; content only — `Scripts/jquery-ui-1.8.11.js` plus the nested theme tree under `Content/` | `:6` |
| nuget.org | EntityFramework | `4.1.10331.0` | ORM, and **the only pin in this edition whose payload the project references**: `<HintPath>..\packages\EntityFramework.4.1.10331.0\lib\EntityFramework.dll</HintPath>` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:37-40], bound as `EntityFramework, Version=4.1.0.0` | `:7` |
| nuget.org | Modernizr | `1.7` | Browser feature detection; content only — `Scripts/modernizr-1.7.js`. The manifest says `1.7` — two parts | `:8` |

Note the `EntityFramework` row's two different numbers, both correct and both transcribed: the **package** version is `4.1.10331.0` (the folder name and the pin), while the **assembly** version the project binds is `4.1.0.0` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:37]. Package version and assembly version are not the same identifier, and §4.4 collects every place in this repository where they diverge.

#### 3.3.1 Five of six pins are content, and the framework itself is not a pin

Only `EntityFramework` is referenced as an assembly; the other five deliver static files. MVC 3's *application* framework — ASP.NET MVC 3 itself — is not in this manifest at all. It is a machine-installed dependency, and §4.1 records it.

MVC 3's `Scripts/` folder makes the same point from the other direction. It ships `jquery.validate.unobtrusive.js`, `jquery.unobtrusive-ajax.js`, `MicrosoftAjax.js` and `MicrosoftMvcValidation.js` with **no corresponding pin of any kind** — those files arrive from the ASP.NET MVC 3 project template, not from a package:

```bash
git ls-files 'src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Scripts/*'
```

So MVC 3's six-line manifest understates its dependency surface more than either other edition's does.

---

## 4. Dependencies NuGet does not resolve

The 63 pins of §3 are not the dependency surface. Each edition also depends on assemblies and products that no manifest declares: .NET Framework assemblies from the targeting pack, assemblies installed machine-wide by a separate product, a native database provider, and assembly versions declared only in configuration. This section catalogues them, because an inventory that stopped at `packages.config` would let a migration plan discover them at build time instead.

### 4.1 Finding — MVC 3's MVC framework assembly is not a pin, and its provider is not a package

Two of MVC 3's dependencies are entirely undeclared by any manifest, and both are required to build or run it.

**`System.Web.Mvc` is referenced with no `HintPath`.** The reference is `<Reference Include="System.Web.Mvc, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35, processorArchitecture=MSIL" />` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:42] — a bare, self-closing reference with no hint path and no package folder behind it. It therefore resolves only from a **machine-wide install of the ASP.NET MVC 3 Tools Update**, a separately installed, out-of-support product. That is a first-class dependency of this edition that appears in no `packages.config`, and it must be recorded as such.

Its immediate neighbours are the same class of dependency:

| Reference | Locator | Resolution |
| --- | --- | --- |
| `System.Data.Entity` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:41] | .NET Framework assembly (targeting pack) |
| `System.Web.Mvc, Version=3.0.0.0` | [:42] | **machine-wide ASP.NET MVC 3 Tools Update — no package** |
| `System.Web.WebPages, Version=1.0.0.0` | [:43] | machine-wide, same product — no package |
| `System.Web.Helpers, Version=1.0.0.0` | [:44] | machine-wide, same product — no package |
| `System.Web.Entity` | [:50] | .NET Framework assembly (targeting pack) |

The project's configuration corroborates all three of the machine-installed assemblies rather than contradicting them. `web.config` lists five assemblies explicitly for the ASP.NET compilation system [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:17-23] — `System.Web.Abstractions` `4.0.0.0`, `System.Web.Helpers` `1.0.0.0`, `System.Web.Routing` `4.0.0.0`, `System.Web.Mvc` `3.0.0.0` and `System.Web.WebPages` `1.0.0.0` — and adds a binding redirect that pins any `System.Web.Mvc` reference between `1.0.0.0` and `2.0.0.0` forward to `3.0.0.0` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:50-51]. A further version declaration sits in app settings: `webpages:Version` = `1.0.0.0` [:9].

**MVC 3's database provider is also not a package.** Its only connection string declares `providerName="System.Data.SqlServerCe.4.0"` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:55-59] — SQL Server Compact 4.0, a separately installed product with a native component. No pin delivers it, no `HintPath` references it, and no `.sdf` data file is committed. As a dependency-inventory fact: **running MVC 3 requires a machine-wide install of SQL Server Compact 4.0 in addition to the MVC 3 Tools Update.** Whether that provider has a forward path is a blocker question owned by [12 — Migration Blockers](12-migration-blockers.md), and the database components each edition needs in order to run are owned by [10 — Build and Deployment Requirements](10-build-and-deployment-requirements.md).

### 4.2 Framework assembly references per edition

Every edition also depends on .NET Framework assemblies resolved from the targeting pack rather than from any package. Counting `<Reference>` elements against those carrying a `<HintPath>`:

| Edition | `<Reference>` elements | With a `<HintPath>` | Without — resolved from the framework or machine-wide | Evidence |
| --- | --- | --- | --- | --- |
| MVC 5 | 46 | 26 | **20** | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:47-63], [:68-71], [:103] |
| MVC 4 | 47 | 24 | **23** | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:44-64], [:75-76], [:80-81] |
| MVC 3 | 24 | 1 | **23** | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:41-63] |

```bash
for f in src/MVC5/MvcMusicStore/MvcMusicStore.csproj \
         src/MVC4/MvcMusicStore/MvcMusicStore.csproj \
         src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj; do
  printf '%s refs=%s hintpaths=%s\n' "$f" "$(grep -c '<Reference Include' "$f")" "$(grep -c '<HintPath>' "$f")"
done
```

Two of MVC 4's twenty-three unhinted references are the `System.Net.Http` pair discussed in §3.2.2 — declared without a hint path even though a pin exists that could have supplied them, which is why the counts here and the pin counts in §3 answer different questions and must not be added together. MVC 3's twenty-three include the three machine-installed assemblies of §4.1, which are *not* framework assemblies; the column is therefore titled "framework or machine-wide" rather than "framework".

The consequence for build prerequisites — which targeting pack, and which separately installed product, each edition needs — is owned by [10 — Build and Deployment Requirements](10-build-and-deployment-requirements.md).

### 4.3 Build-tooling dependencies declared in the project files

These are dependencies in the operational sense: without them MSBuild does not evaluate the project. They are inventoried here and their build consequences are owned by deliverable 10.

| Edition | Import | Locator | Condition |
| --- | --- | --- | --- |
| MVC 5 | `$(MSBuildBinPath)\Microsoft.CSharp.targets` | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:271] | unconditional |
| MVC 5 | `$(VSToolsPath)\WebApplications\Microsoft.WebApplication.targets` | [:272] | `'$(VSToolsPath)' != ''` |
| MVC 5 | `$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v10.0\WebApplications\Microsoft.WebApplication.targets` | [:273] | `false` — inert |
| MVC 5 | `$(SolutionDir)\.nuget\NuGet.targets` | [:295] | `Exists(...)` — **conditional** |
| MVC 4 | `$(VSToolsPath)\WebApplications\Microsoft.WebApplication.targets` | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:337] | `'$(VSToolsPath)' != ''` |
| MVC 4 | `$(SolutionDir)\.nuget\nuget.targets` | [:360] | **none — unconditional** |
| MVC 3 | `$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v10.0\WebApplications\Microsoft.WebApplication.targets` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:209] | **none — unconditional** |

MVC 4 additionally opts into restore-on-build: `<SolutionDir Condition="$(SolutionDir) == '' Or $(SolutionDir) == '*Undefined*'">..\</SolutionDir>` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:23] and `<RestorePackages>true</RestorePackages>` [:24]. That single property is what makes the committed NuGet client of §5 an active build dependency rather than a dormant file.

### 4.4 Assembly versions declared only in configuration, and where they diverge from the pin

Both MVC 5 and MVC 4 carry `assemblyBinding` redirects and an `entityFramework` section that name assembly versions. These are dependency declarations in their own right: they are what the runtime honours, and they are not always the same number as the package pin.

**MVC 5** [src/MVC5/MvcMusicStore/Web.config:41-60]:

| Assembly | Redirect | Pin that supplies it | Same number? |
| --- | --- | --- | --- |
| `System.Web.Helpers` | `1.0.0.0-3.0.0.0` → `3.0.0.0` [:44-45] | Microsoft.AspNet.WebPages `3.0.0` | yes |
| `System.Web.Mvc` | `1.0.0.0-5.0.0.0` → `5.0.0.0` [:48-49] | Microsoft.AspNet.Mvc `5.0.0` | yes |
| `System.Web.WebPages` | `1.0.0.0-3.0.0.0` → `3.0.0.0` [:52-53] | Microsoft.AspNet.WebPages `3.0.0` | yes |
| `WebGrease` | `1.0.0.0-1.5.2.14234` → **`1.5.2.14234`** [:56-57] | WebGrease **`1.5.2`** | **no** |

Also in MVC 5: the `entityFramework` configuration section declares `EntityFramework, Version=6.0.0.0` [src/MVC5/MvcMusicStore/Web.config:9], matching the EntityFramework `6.0.0` pin, and the EF provider registration names `EntityFramework.SqlServer` [:68] — the second assembly from that same pin. `webpages:Version` is `3.0.0.0` [:18].

**MVC 4** [src/MVC4/MvcMusicStore/Web.config:65-74]: `System.Web.Helpers` → `2.0.0.0`, `System.Web.Mvc` → `4.0.0.0`, `System.Web.WebPages` → `2.0.0.0`; the `entityFramework` section declares `EntityFramework, Version=5.0.0.0` [:9]; `webpages:Version` is `2.0.0.0` [:27].

**MVC 3** is covered in §4.1: five explicit assemblies [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:17-23], one redirect to `3.0.0.0` [:50-51], `webpages:Version` `1.0.0.0` [:9].

The four places in this repository where a package version and the corresponding assembly version are different numbers, collected so no downstream document has to rediscover them:

| Package pin | Assembly version | Locator |
| --- | --- | --- |
| WebGrease `1.5.2` (MVC 5) | `1.5.2.14234` | [src/MVC5/MvcMusicStore/Web.config:57] |
| EntityFramework `4.1.10331.0` (MVC 3) | `4.1.0.0` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:37] |
| Microsoft.AspNet.Mvc `4.0.20710.0` (MVC 4) | `4.0.0.0` | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:92] |
| Microsoft.AspNet.WebPages `2.0.20710.0` (MVC 4) | `2.0.0.0` | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:82], [:103], [:107], [:111] |

The same divergence holds for the other `20710`-stamped MVC 4 packages, whose assemblies all bind at two-part-derived `4.0.0.0` or `2.0.0.0` versions. The general rule this repository demonstrates: **the version to quote when identifying a package is the `packages.config` value; the version to quote when reasoning about binding is the assembly version, and in this repository they differ more often than they agree.**

---

## 5. The restore client is itself a committed, pinned dependency

### 5.1 A 2012-era NuGet client is tracked in source control

`src/MVC4/MvcMusicStore/.nuget/NuGet.exe` is a tracked binary, not a build artifact. Its verified properties:

| Property | Value |
| --- | --- |
| Size | **630,784 bytes** |
| `FileVersion` | **2.0.30828.5** |
| Assembly version | **2.0.30828.5** |
| `ProductVersion` | **2.0.40001** |
| `ProductName` | NuGet |
| `CompanyName` | Outercurve Foundation |
| SHA-256 | `E52E94A96B7D9F8C1DF5154297468F8FD0260331FB9DE1D48EE6C5867FDD1C09` |

Reproduce on any host with PowerShell:

```powershell
$p = 'src/MVC4/MvcMusicStore/.nuget/NuGet.exe'
(Get-Item $p).Length                                                  # -> 630784
[System.Diagnostics.FileVersionInfo]::GetVersionInfo((Resolve-Path $p)) | Format-List FileVersion, ProductVersion, ProductName, CompanyName
[System.Reflection.AssemblyName]::GetAssemblyName((Resolve-Path $p)).Version   # -> 2.0.30828.5
(Get-FileHash $p -Algorithm SHA256).Hash
```

This is a **NuGet 2.0 client from 2012**, and it is a dependency in its own right for two reasons. First, it is what MVC 4's restore actually executes: `NuGet.targets` resolves `<NuGetExePath ... >$(NuGetToolsPath)\nuget.exe</NuGetExePath>` [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:43] and builds its restore invocation around it [:53]. Second, the fallback that would replace it is switched off — `<DownloadNuGetExe Condition=" '$(DownloadNuGetExe)' == '' ">false</DownloadNuGetExe>` [:16] — so the committed file is required rather than optional.

It is the MSBuild-integrated restore mechanism, activated by `<RestorePackages>true</RestorePackages>` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:24], which is a distinct and long-deprecated mechanism from SDK-integrated restore: NuGet dropped MSBuild-integrated restore with the NuGet 3 generation. Two further properties of that wiring belong to the inventory: restore consent is required — `<RequireRestoreConsent Condition=" '$(RequireRestoreConsent)' != 'false' ">true</RequireRestoreConsent>` [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:13], which adds `-RequireConsent` to the restore command [:51] — and the restore command's package output directory is `$(SolutionDir)\packages` [:31], which is how `SolutionDir` comes to determine where MVC 4's packages land.

The consequence for building MVC 4, including how `SolutionDir` must be supplied, is owned by [10 — Build and Deployment Requirements](10-build-and-deployment-requirements.md) and is not restated here. As a dependency fact: **building MVC 4 as committed depends on a 2012 executable stored in the repository, and on a restore mechanism that current NuGet no longer supports.**

### 5.2 The restore configuration is present in one edition and only referenced in another

Exactly three files constitute the repository's entire NuGet tooling surface, all under one edition:

```bash
git ls-files | grep '\.nuget/'
# src/MVC4/MvcMusicStore/.nuget/NuGet.Config
# src/MVC4/MvcMusicStore/.nuget/NuGet.exe
# src/MVC4/MvcMusicStore/.nuget/NuGet.targets
```

MVC 5's solution nevertheless declares a `.nuget` solution folder and lists two of those files as its contents — `Project("{2150E333-8FDC-42A3-9474-1A3956D46DE8}") = ".nuget", ".nuget", ...` [src/MVC5/MvcMusicStore.sln:8] with `.nuget\NuGet.Config` and `.nuget\NuGet.targets` [:10-11] — and MVC 5's project imports `$(SolutionDir)\.nuget\NuGet.targets` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:295]. **No such folder or files exist under `src/MVC5`.** The import is guarded by `Exists(...)`, so it is skipped rather than failing, but the effect is that MVC 5 declares restore configuration it does not have and consequently has no restore wiring of its own at all. MVC 3 declares none and has none.

`NuGet.exe` and the two committed `packages/` trees are also excluded by `.gitignore` yet tracked; that combination, and its severity, is owned by [08 — Technical Debt Register](08-technical-debt-register.md) (§7.2 records the dependency-side facts).

---

## 6. The restore source is not configured anywhere — and a correction to Technical Specification §3.3

### 6.1 What the repository actually contains

Three verified facts, and they are the whole of the repository's evidence on this question.

**1 — The only `NuGet.Config` in the repository configures no package source.** The file is six lines end to end:

```xml
<?xml version="1.0" encoding="utf-8"?>          <!-- :1 -->
<configuration>                                 <!-- :2 -->
  <solution>                                    <!-- :3 -->
    <add key="disableSourceControlIntegration" value="true" />   <!-- :4 -->
  </solution>                                   <!-- :5 -->
</configuration>                                <!-- :6 -->
```

Its single setting is `disableSourceControlIntegration` [src/MVC4/MvcMusicStore/.nuget/NuGet.Config:4], which concerns source-control integration and has nothing to do with feeds. **There is no `packageSources` element** — not empty, absent.

**2 — The `nuget.org` v2 endpoint that appears in `NuGet.targets` is inside a comment.** The relevant block:

```xml
<ItemGroup Condition=" '$(PackageSources)' == '' ">                          <!-- :19 -->
    <!-- Package sources used to restore packages. By default will used the registered sources under %APPDATA%\NuGet\NuGet.Config -->   <!-- :20 -->
    <!--                                                                     :21
        <PackageSource Include="https://nuget.org/api/v2/" />                 :22
        <PackageSource Include="https://my-nuget-source/nuget/" />            :23
    -->                                                                      <!-- :24 -->
</ItemGroup>                                                                 <!-- :25 -->
```

The `https://nuget.org/api/v2/` endpoint sits at [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:22], **inside the XML comment spanning [:21-24]**, within an `ItemGroup` that is itself conditioned on `'$(PackageSources)' == ''` [:19]. Two things follow. The `ItemGroup` body is empty, so the `PackageSource` item list is empty. And the second commented line [:23] is a *private-feed* example — equally inert, and a useful reminder that neither line is a configured value.

**3 — No package source is configured anywhere else in the repository, because there is no other configuration file to hold one.**

```bash
git ls-files | grep -iE '(^|/)nuget\.config$'        # -> src/MVC4/MvcMusicStore/.nuget/NuGet.Config, and nothing else
git ls-files 'NuGet.config' 'NuGet.Config' 'nuget.config' | wc -l   # -> 0   (no root-level file)
```

Note also the repository's own comment at [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:20], which documents the behaviour explicitly: with no sources listed, restore uses the registered sources under `%APPDATA%\NuGet\NuGet.Config`. The repository states its own inheritance.

### 6.2 The correction

> **Correction to Technical Specification §3.3.** Specification §3.3 describes the `https://nuget.org/api/v2/` endpoint in `NuGet.targets` as the *configured default* package source for this repository. **The repository does not support that reading.** The endpoint exists only inside a commented-out example block [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:21-24, with the endpoint at :22]; the surrounding `ItemGroup` is empty [:19-25]; and the repository's only `NuGet.Config` contains a single `disableSourceControlIntegration` setting with no `packageSources` element at all [src/MVC4/MvcMusicStore/.nuget/NuGet.Config:4]. Under the citation policy that governs this assessment — repository evidence is primary, and a specification section may be cited alongside it but never instead of it — **the repository governs, and the specification's characterization is corrected here rather than inherited.** No claim in this document rests on that endpoint being configured.

This is recorded as a correction, in these terms, rather than as a discrepancy note, because the two readings lead to materially different conclusions and a reader who saw both cited as agreeing would draw the wrong one.

Specification §3.3 remains a useful secondary cross-reference for other parts of the dependency picture — the `packages.config`-only manifest format with no central version management, the per-edition restore weight, and the governance risk of committed executable content inside package payloads, which this document corroborates independently in §7.2. It is the *configured source* claim specifically that is corrected.

### 6.3 The consequence, which is a finding in its own right

Because the `PackageSource` item list is empty, `NuGet.targets` resolves `<PackageSources Condition=" $(PackageSources) == '' ">@(PackageSource)</PackageSources>` [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:44] to an empty value, and the restore command it constructs — `$(NuGetCommand) install "$(PackagesConfig)" -source "$(PackageSources)" ...` [:53] — is therefore issued with an **empty `-source`**. The client falls back to whatever sources its configuration hierarchy provides.

Stated plainly, and this is the finding:

> **The effective package source set is not knowable from the repository.** Restore inherits whatever machine-level and user-level NuGet configuration the build host happens to carry. It follows that **the repository cannot be asserted to exclude private feeds** — not because there is evidence of one, but because the repository contains no evidence either way. Any statement of the form "this project restores from nuget.org" is a statement about a particular build host, not about this repository, and must be attributed to that host.

Two corollaries worth stating because they are easy to get backwards:

- This does not contradict §2.1. Every one of the 63 **identifiers** is public and verifiable on nuget.org; what is unknowable is which **feed** serves them. An unpinned source combined with public identifiers is precisely the shape in which a substituted package would be hard to detect.
- The finding is about the repository, so it is unaffected by any particular host's configuration being benign. A build performed on a host whose user-level configuration lists only nuget.org is reproducible on that host and nowhere else, and the repository records nothing that would let a second host reproduce it.

The target-state remedy — a committed `NuGet.config` that clears inherited sources and declares its own — is specified by [04 — .NET 8 Migration Strategy](04-dotnet8-migration-strategy.md). This document deliberately does not specify it.

---

## 7. What the repository does not pin

### 7.1 No lockfile exists in any edition

```bash
git ls-files 'packages.lock.json' | wc -l          # -> 0
git ls-files | grep -c 'packages.lock.json'        # -> 0
```

`packages.lock.json` is absent throughout — not stale, not partial, absent. There is also no central version management file of any kind: no `Directory.Build.props`, no `Directory.Packages.props`, no `.ruleset`, no `.editorconfig`.

```bash
git ls-files | grep -iE '(\.ruleset$)|(Directory\.Build\.props)|(Directory\.Packages\.props)|(\.editorconfig)' | wc -l   # -> 0
```

The consequence is precise and worth stating carefully, because the exactness of the 63 pins can make it look as though it does not apply. **Exact direct pins do not lock transitive dependencies.** `packages.config` records the direct graph plus whatever transitive packages a past install happened to flatten into the same file; it carries no resolved transitive graph, no version ranges resolved to selections, and no content hashes. Two restores of the same manifest against different source configurations, or the same source at different times, can produce different transitive payloads, and nothing in the repository would detect it.

So the repository's reproducibility position, stated as it actually is: the **direct** dependency set is fully pinned and byte-verifiable from the manifests; the **transitive** set is not pinned, not recorded and not reproducible from the repository; and the **source** those pins resolve from is not specified at all (§6). This is supply-chain debt — its severity, remediation and owner are assigned by [08 — Technical Debt Register](08-technical-debt-register.md), and the target-side locking decision is owned by [04 — .NET 8 Migration Strategy](04-dotnet8-migration-strategy.md).

### 7.2 Two editions commit their restored packages; one commits nothing

```bash
git ls-files | grep -c '/packages/'    # -> 215
```

**215 tracked files** sit under the two committed `packages/` trees, despite `.gitignore:15` declaring `packages/*` (with `Packages/` additionally at `.gitignore:33`). The distribution:

| Edition | Tracked files under `packages/` | Package folders | Matches its pin count? | `.dll`/`.exe` files |
| --- | --- | --- | --- | --- |
| MVC 4 — `src/MVC4/MvcMusicStore/packages/` | 169 | 29, plus `repositories.config` | **yes** — all 29 pins present at their exact pinned versions | 31 |
| MVC 3 — `src/MVC3/MvcMusicStore-Completed/packages/` | 46 | 6, plus `repositories.config` | **yes** — all 6 pins present at their exact pinned versions | 1 |
| MVC 5 | **0** | — | n/a | 0 |

```bash
git ls-files 'src/MVC4/MvcMusicStore/packages/*' | wc -l                 # -> 169
git ls-files 'src/MVC3/MvcMusicStore-Completed/packages/*' | wc -l       # -> 46
git ls-files | grep '/packages/' | grep -cE '\.(dll|exe)$'               # -> 32
```

Three dependency-relevant observations, kept separate from the debt framing that deliverable 08 owns:

- **The committed payloads corroborate the pins exactly.** Every package folder name is `<id>.<version>` at the version its manifest declares, in both editions. This is a second, independent confirmation of the version strings in §3.2 and §3.3 — the manifest and the on-disk payload agree, so a transcription error in this document would be caught twice.
- **32 committed `.dll`/`.exe` files sit inside those payloads.** Executable content is tracked in source control and is not covered by any lockfile or hash manifest. Technical Specification §3.3 identifies the same governance risk, cited here as a secondary cross-reference alongside the repository evidence above.
- **MVC 5, the sole migration source, commits nothing.** It therefore cannot be built from a clean checkout without a working restore — and, per §6, without a source that the repository does not specify. The build consequence is owned by [10 — Build and Deployment Requirements](10-build-and-deployment-requirements.md).

---

## 8. Version-risk posture

### 8.1 The finding, stated as a class

The client-side and OAuth pins in this repository are 2011–2013 releases. The exact pins in that group, with the manifest that declares each:

| Package | Version | Manifest |
| --- | --- | --- |
| DotNetOpenAuth.AspNet, .Core, .OAuth.Consumer, .OAuth.Core, .OpenId.Core, .OpenId.RelyingParty | `4.0.3.12153` | [src/MVC4/MvcMusicStore/packages.config:3-8] |
| Newtonsoft.Json | `4.5.6` | [src/MVC4/MvcMusicStore/packages.config:30] |
| Newtonsoft.Json | `5.0.6` | [src/MVC5/MvcMusicStore/packages.config:27] |
| jQuery | `1.5.1` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config:3] |
| jQuery | `1.7.1.1` | [src/MVC4/MvcMusicStore/packages.config:10] |
| jQuery | `1.10.2` | [src/MVC5/MvcMusicStore/packages.config:6] |
| jQuery.UI.Combined | `1.8.11` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config:6] |
| jQuery.UI.Combined | `1.8.20.1` | [src/MVC4/MvcMusicStore/packages.config:11] |
| bootstrap | `3.0.0` | [src/MVC5/MvcMusicStore/packages.config:4] |

Two aggravating factors are repository facts rather than inferences: every one of these client-side libraries is **self-hosted from the application's own `Scripts/` and `Content/` directories** rather than loaded from a versioned CDN (§3.1.4, §3.2.4), and none is served with a **Subresource Integrity** attribute or under a **Content Security Policy** — the repository declares neither anywhere.

Three further pins belong to the same era and the same class even though they are not client-side: `Owin` `1.0`, the five `Microsoft.Owin*` `2.0.0` packages [src/MVC5/MvcMusicStore/packages.config:16-24], and `Microsoft.Web.Infrastructure` `1.0.0.0`, whose four-part version reflects its 2012 origin.

### 8.2 Observed advisory evidence, and exactly what it is

A scan result exists and it is not one this document had to go looking for: **NuGet's own restore-time audit emits it, and it is reproduced by simply restoring each solution.** Restoring all three editions with NuGet **6.11.1.2** on **2026-08-27** produced **43 advisory warnings** — `NU1902` for moderate severity and `NU1903` for high — naming GitHub advisory identifiers against **14 of the 63 pins**. Restore still succeeded: all three exit `0`, and these are warnings, not errors.

```bash
nuget restore src/MVC5/MvcMusicStore.sln -NonInteractive   # then MVC4 and MVC3; grep the output for NU19
# each run: exit 0, with NU1902/NU1903 warnings on the counts below
```

| Edition | Warnings | High (`NU1903`) | Moderate (`NU1902`) | Pins named |
| --- | --- | --- | --- | --- |
| MVC 5 | 15 | 6 | 9 | `bootstrap` `3.0.0`; `jQuery` `1.10.2`; `jQuery.Validation` `1.11.1`; `Microsoft.AspNet.Identity.Owin` `1.0.0`; `Microsoft.Owin` `2.0.0`; `Microsoft.Owin.Security.Cookies` `2.0.0`; `Newtonsoft.Json` `5.0.6` |
| MVC 4 | 14 | 2 | 12 | `jQuery` `1.7.1.1`; `jQuery.UI.Combined` `1.8.20.1`; `jQuery.Validation` `1.9.0.1`; `Newtonsoft.Json` `4.5.6` |
| MVC 3 | 14 | 1 | 13 | `jQuery` `1.5.1`; `jQuery.UI.Combined` `1.8.11`; `jQuery.Validation` `1.8.0` |
| **Total** | **43** | **9** | **34** | **14 distinct pins, 8 distinct package identifiers** |

**This document reports those identifiers only because a tool emitted them, verbatim, in output any reader can regenerate. It has looked up, inferred, matched or constructed no identifier of its own** — that distinction is the whole of the discipline here, and an inventory whose other numbers are exact cannot afford one fabricated identifier.

Three of the observed results are worth pulling out, because each connects to a finding stated earlier on independent grounds:

- **`Microsoft.Owin.Security.Cookies` `2.0.0` carries a high-severity advisory, and it is the one authentication middleware the application actually enables** (§3.1.2). `Microsoft.Owin` `2.0.0` and `Microsoft.AspNet.Identity.Owin` `1.0.0` are flagged high as well. This is the most operationally significant group in the table: it is on the live sign-in path, not in dormant scaffolding.
- **`Newtonsoft.Json` carries a high-severity advisory in both editions — `5.0.6` and `4.5.6` — and §3.1.3 established that no application source calls it.** A package with no consumer is nonetheless restored, copied to `bin` and deployed, so it contributes exposure and delivers nothing. That makes its removal the cheapest item in the table.
- **`bootstrap` `3.0.0`, `jQuery` and `jQuery.UI.Combined` account for 28 of the 34 moderate warnings**, and per §3.1.4 and §3.2.4 those are precisely the packages whose payloads are already **vendored into the tree** — so the flagged code is committed to source control and served to browsers today, without SRI or a CSP.

**What this evidence is not**, stated plainly so nobody over-reads it:

- It is an audit of the **direct pins declared in the three manifests**, not a full software-composition analysis. Per §7.1 the transitive graph is unpinned, so it describes neither transitive exposure nor what a future restore would resolve.
- It is **time-varying**. The advisory database moves, so the counts above are the result *on the stated date with the stated client*, not a stable property of the repository. Re-running it later is expected to give a different answer, which is why the date and the client version are recorded with it.
- It **cannot see the vendored copies** under `Scripts/`, `Content/` and `fonts/`. A package audit inspects packages; the committed asset files are ordinary tracked content, and no package audit reaches them. The vendored copies match the pinned versions (§3.1.4), so the exposure carries over — but that inference is mine and the audit does not make it.
- It **says nothing about the non-NuGet dependencies of §4.1** — the machine-wide ASP.NET MVC 3 Tools Update and SQL Server Compact 4.0 install — which no package audit can assess.
- And it is **not a dependency-scanning capability the repository possesses.** The repository has no scanning configuration whatsoever, verified:

```bash
git ls-files | grep -c '^\.github/'                                  # -> 0   (no GitHub workflows or config directory)
git ls-files | grep -icE 'dependabot|renovate'                       # -> 0   (no Dependabot, no Renovate)
git ls-files '*packages.config' | grep -v '/packages/' \
  | xargs grep -hiE 'analyzer|StyleCop|FxCop|SonarAnalyzer|Roslynator' | wc -l   # -> 0   (no analyzer package in any manifest)
git ls-files | grep -iE '(\.ruleset$)|(Directory\.Build\.props)'     # -> (no output)
```

That last point is the finding that survives whatever the advisory counts happen to be on a given day: **the evidence above is visible only to whoever happens to read restore output, and nothing in this repository records it, gates on it or fails a build because of it.** Forty-three warnings scroll past on every clean restore and no artifact retains them.

### 8.3 What must still happen

The audit above is a floor, not a substitute for a scan. **Directive for the implementation phase:** run a full software-composition analysis against the restored graph — direct *and* transitive — on a host with network access, extend it to the vendored client-side assets that no package audit inspects, record its dated output with the tool and version that produced it, and add scanning configuration to the repository so the result is gated rather than merely printed. Any advisory identifier quoted downstream must come from that recorded output or from the reproducible restore audit above, never from recollection.

The output is an input to the risk register owned by [07 — Effort, Risks and Sequencing](07-effort-risks-sequencing.md), and the roadmap gate at which it runs is owned by [03 — Modernization Roadmap](03-modernization-roadmap.md). The eventual disposition of each of these pins — updated, replaced or removed — is owned by [04 — .NET 8 Migration Strategy](04-dotnet8-migration-strategy.md), and this document names no successor for any of them.

---

## 9. Summary of findings

Each row is an inventory finding of this document; the deliverable named in the last column carries its severity, remediation, effort or forward decision.

| # | Finding | Evidence | Consequence owned by |
| --- | --- | --- | --- |
| F-02-01 | 63 exact pins across three manifests, all public nuget.org identifiers, no ranges, no private package | §2, §3 | — (this document is the owner) |
| F-02-02 | MVC 5's manifest declares `net45` for all 28 pins while the project targets `v4.8` | §3.1.1 | 04, 10 |
| F-02-03 | Four dormant external-login provider packages ship in MVC 5; the fifth `Security.*` package is OAuth infrastructure, not a provider | §3.1.2 | 04, 08 |
| F-02-04 | `Newtonsoft.Json` is deployed in both MVC 5 and MVC 4 but called from no application source | §3.1.3 | 04 (removal), 12 (serialization) |
| F-02-05 | Six of MVC 5's pins, seven of MVC 4's and five of MVC 3's deliver content, not assemblies | §3.1.4, §3.2, §3.3 | 05 |
| F-02-06 | MVC 4's `20710` release stamp spans two different version strings across thirteen pins | §3.2.1 | 04 |
| F-02-07 | Four Web API packages serve a mapped route with zero `ApiController` implementations; one is a metapackage with no assembly and one is inert on `net45` | §3.2.2 | 08 |
| F-02-08 | Six DotNetOpenAuth packages plus the WebPages OAuth surface serve four commented-out registrations | §3.2.3 | 04, 08 |
| F-02-09 | MVC 4 depends on `Antlr3.Runtime` with no `Antlr` pin, taking it from the WebGrease payload | §3.2.5 | 04 |
| F-02-10 | MVC 3's six pins carry no `targetFramework` attribute at all | §3.3 | 10 |
| F-02-11 | MVC 3 depends on a machine-wide ASP.NET MVC 3 Tools Update install — `System.Web.Mvc` `3.0.0.0` is referenced with no `HintPath` and no pin | §4.1 | 10, 12 |
| F-02-12 | MVC 3 additionally depends on a machine-wide SQL Server Compact 4.0 install, declared only as a `providerName` | §4.1 | 10, 12 |
| F-02-13 | 20 / 23 / 23 references per edition resolve from the framework or machine-wide rather than from any package | §4.2 | 10 |
| F-02-14 | Package version and assembly version differ in four identified places, including WebGrease `1.5.2` binding to `1.5.2.14234` | §4.4 | 04 |
| F-02-15 | A 2012 NuGet 2.0 client (630,784 bytes, `2.0.30828.5`) is committed and is required by MVC 4's restore, with the download fallback disabled | §5.1 | 08, 10 |
| F-02-16 | MVC 5's solution declares a `.nuget` folder that does not exist; MVC 5 has no restore wiring of its own | §5.2 | 10 |
| F-02-17 | **No package source is configured anywhere.** Technical Specification §3.3 is corrected: the v2 endpoint is inside a comment | §6.1, §6.2 | 04 |
| F-02-18 | The effective source set is not knowable from the repository, and private feeds cannot be ruled out | §6.3 | 04, 08 |
| F-02-19 | No lockfile and no central version management in any edition; transitive resolution is not reproducible | §7.1 | 08, 04 |
| F-02-20 | 215 tracked files, including 32 `.dll`/`.exe`, committed under two `packages/` trees despite `.gitignore:15`; MVC 5 commits none | §7.2 | 08, 10 |
| F-02-21 | A class of aged (2011–2013), self-hosted, SRI- and CSP-unprotected dependencies, enumerated by exact pin | §8.1 | 07 (risk), 04 (disposition) |
| F-02-22 | NuGet's restore audit names **14 of the 63 pins** across 43 `NU1902`/`NU1903` warnings, 9 of them high severity — including the one authentication middleware actually enabled and the package no application source calls | §8.2 | 07 (risk), 04 (disposition) |
| F-02-23 | Nothing in the repository records, retains or gates on that audit output; 43 warnings scroll past on every clean restore and no artifact keeps them | §8.2, §8.3 | 03 (gate), 08 (severity) |

---

## 10. Reproducibility appendix

Every command this document quotes, in one place. All are read-only **except** the three `nuget restore` invocations that produce the §8.2 audit evidence, which are flagged where they appear. The canonical forms are POSIX per AAP 0.11.3; on the Windows verification host they were run through the bundled Git-for-Windows `bash` from the repository root, and the values shown are the values observed.

```bash
# --- The headline count -------------------------------------------------------
git ls-files '*packages.config' | grep -v '/packages/' | xargs grep -h '<package ' | wc -l   # 63

# --- Per-edition pin counts and the exact strings ----------------------------
for f in src/MVC5/MvcMusicStore/packages.config \
         src/MVC4/MvcMusicStore/packages.config \
         src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config; do
  printf '%s pins=%s net45=%s tfattr=%s\n' "$f" \
    "$(grep -c '<package ' "$f")" \
    "$(grep -c 'targetFramework=\"net45\"' "$f")" \
    "$(grep -c 'targetFramework=' "$f")"
  grep -o 'id="[^"]*" version="[^"]*"' "$f"
done
# MVC5 pins=28 net45=28 tfattr=28   MVC4 pins=29 net45=29 tfattr=29   MVC3 pins=6 net45=0 tfattr=0

# --- Project target frameworks ----------------------------------------------
grep -n 'TargetFrameworkVersion' src/MVC5/MvcMusicStore/MvcMusicStore.csproj \
  src/MVC4/MvcMusicStore/MvcMusicStore.csproj \
  src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj      # v4.8 / v4.5 / v4.0

# --- References vs HintPaths (framework and machine-wide dependencies) -------
for f in src/MVC5/MvcMusicStore/MvcMusicStore.csproj \
         src/MVC4/MvcMusicStore/MvcMusicStore.csproj \
         src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj; do
  printf '%s refs=%s hintpaths=%s\n' "$f" "$(grep -c '<Reference Include' "$f")" "$(grep -c '<HintPath>' "$f")"
done                                                                        # 46/26, 47/24, 24/1

# --- NuGet tooling and the source configuration ------------------------------
git ls-files | grep '\.nuget/'                                              # 3 files, MVC4 only
git ls-files | grep -iE '(^|/)nuget\.config$'                               # 1 file, MVC4 only
git ls-files 'NuGet.config' 'NuGet.Config' 'nuget.config' | wc -l           # 0 at root
sed -n '19,25p' src/MVC4/MvcMusicStore/.nuget/NuGet.targets                 # the commented example block

# --- What is not pinned ------------------------------------------------------
git ls-files 'packages.lock.json' | wc -l                                   # 0
git ls-files | grep -c 'packages.lock.json'                                 # 0
git ls-files | grep -iE '(\.ruleset$)|(Directory\.Build\.props)|(Directory\.Packages\.props)|(\.editorconfig)' | wc -l   # 0

# --- Committed package payloads ---------------------------------------------
git ls-files | grep -c '/packages/'                                         # 215
git ls-files 'src/MVC4/MvcMusicStore/packages/*' | wc -l                    # 169
git ls-files 'src/MVC3/MvcMusicStore-Completed/packages/*' | wc -l          # 46
git ls-files | grep '/packages/' | grep -cE '\.(dll|exe)$'                  # 32
grep -in 'packages' .gitignore                                              # :15 packages/*  ; :33 Packages/  (-i is required: :33 is capitalised)

# --- Dead scaffolding and unused dependencies -------------------------------
git ls-files '*.cs' '*.cshtml' | grep -v '/packages/' | xargs grep -lE 'Newtonsoft|JsonConvert' | wc -l   # 0
git ls-files '*.cs' | grep -v '/packages/' | xargs grep -n 'ApiController' | wc -l                        # 0

# --- The restore audit of §8.2 (the one command here that is NOT read-only:
#     it writes packages/ and requires network access; run it in a scratch clone)
nuget restore src/MVC5/MvcMusicStore.sln -NonInteractive        # exit 0, 15 NU1902/NU1903 warnings
nuget restore src/MVC4/MvcMusicStore.sln -NonInteractive        # exit 0, 14
nuget restore src/MVC3/MvcMusicStore-Completed/MvcMusicStore.sln -NonInteractive   # exit 0, 14

# --- Dependency-scanning configuration (all absent) -------------------------
git ls-files | grep -c '^\.github/'                                         # 0
git ls-files | grep -icE 'dependabot|renovate'                              # 0
git ls-files '*packages.config' | grep -v '/packages/' \
  | xargs grep -hiE 'analyzer|StyleCop|FxCop|SonarAnalyzer|Roslynator' | wc -l                            # 0

# --- Asset attribution ------------------------------------------------------
git ls-files 'src/MVC4/MvcMusicStore/Content/*' | wc -l                     # 55
git ls-files 'src/MVC4/MvcMusicStore/Content/themes/*' | wc -l              # 54
git ls-files 'src/MVC5/MvcMusicStore/Scripts/*' 'src/MVC5/MvcMusicStore/Content/*' 'src/MVC5/MvcMusicStore/fonts/*'

# --- The constraint this work was held to -----------------------------------
git status --porcelain            # only new files under docs/modernization/
```

---

*End of deliverable 02. Index and requirement map: [README](README.md). No repository file outside `docs/modernization/` was created, modified or deleted in the production of this document.*
