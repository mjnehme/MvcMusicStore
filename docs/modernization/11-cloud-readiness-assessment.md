# 11 — Cloud Readiness Assessment

## 1. Purpose, scope and authoring contract

### 1.1 What this document is

An assessment of whether the three ASP.NET MVC applications in this repository can run on a managed cloud platform as they stand, and of what must change before they can. It is one of the five supporting assessment records. It consumes deliverable [01 — Architecture Overview](01-architecture-overview.md) and deliverable [02 — Dependency Inventory](02-dependency-inventory.md), and it feeds deliverable [12 — Migration Blockers](12-migration-blockers.md) alongside deliverables 09 and 10.

Its organizing rule is that **every blocker states its target-state replacement, not only the problem**. A cloud readiness assessment that stops at the obstacle tells a reader what is wrong and leaves them no further forward; each of the eight blockers in section 3 therefore closes with a named replacement and a pointer to the deliverable that owns the design detail.

### 1.2 What this document is not

It is not a hosting recommendation, and it is not a design. It changes nothing, provisions nothing, and decides nothing that another deliverable owns. Section 1.4 lists the four decisions that are deliberately **not** made here.

It is also not a security assessment. Several findings below touch the same evidence deliverable 09 examines — the connection strings, the plaintext administrator credential, the absent transport protection — but the question asked here is different: not "is this safe?" but "does this work when the filesystem is ephemeral, the instance count is greater than one, and there is no domain controller?" Where the two overlap, the security consequence is cross-referenced to 09 and not restated.

### 1.3 The no-modification constraint

The user directed **"Do not make code changes initially"**, and the environment setup instructions attached to this project independently restate the same gate: *"Do not modify code until assessment and modernization plan are approved."* Two inputs agreeing on it is why no defect named below is repaired here.

No Azure resource is provisioned by this work — no resource group, App Service plan, Container Apps environment, Azure SQL instance, Key Vault or managed identity. **This is a document.** Every target-state replacement below is a *specification* for a separately approved implementation phase, not an action taken. Naming the replacement is required of this document; creating it is forbidden to it, and both halves matter.

### 1.4 Authoring contract, and the absence of user rules

`review_rules` returns exactly **"No user rules provided."** There is therefore no project rule to name, summarize or comply with, and none is invented in its place. The absence is not licence to lower the bar; this document is held to enterprise-standard assessment practice and to four contracts that stand where rules would:

1. **Every as-is claim carries an inline `[<path>:<locator>]` citation** at the point the claim is made, repository-relative and resolving in the checkout. There is no trailing reference list, because a citation collected at the end cannot be checked against the sentence it supports.
2. **A repository-wide claim carries its reproducing command** adjacent to the claim. Most of the findings below are *absences* — no key configuration, no security header, no logging, no filesystem write — and an absence has no line to point at. The command is its evidence, and it is the stronger form because a reader can re-run it. Section 6 collects them all.
3. **Repository evidence is primary.** The Technical Specification is cited only as a secondary cross-reference, alongside a repository citation and never instead of one.
4. **Every claim names the edition or editions it holds in.** The repository ships three independent applications, not one application with three configurations (deliverable 01 §1.5). A claim about "the application" without an edition qualifier would be unverifiable here.

**Reproducing the commands on this host.** The commands quoted throughout are POSIX shell forms. The verification host is Windows; they were executed through the Git-for-Windows `bash` bundled on the host, from the repository root, and the outputs quoted are the outputs observed there. One practical note, because it changes how a byte total must be computed: **`bc` is not installed on this host**, so every sum below is taken with `awk '{s+=$1} END {print s}'`.

### 1.5 What this document does not own

Four decisions are owned elsewhere. This document states the *requirement* and its *consequence*, then points at the owner. This is not deference for its own sake — a requirement restated in different words downstream reads as a second decision, and a reader who finds two picks one.

| Decision | Owner | What this document does instead |
| --- | --- | --- |
| **Hosting target and deployment model** | [06 — Azure Hosting Recommendations](06-azure-hosting-recommendations.md) | Names the primary and secondary targets by cross-reference only. The choice is **not re-argued here**, and nothing below should be read as reopening it. |
| **The observability and telemetry mechanism** | [06](06-azure-hosting-recommendations.md) | Records the *absence* of any instrumentation today (section 3.8.3) and states that a health surface and structured logging are required. It does **not** name the collection approach. |
| **Where the data-protection key ring lives**, its protect-at-rest behaviour, rotation policy and slot/revision isolation | [06](06-azure-hosting-recommendations.md) | States the requirement for persistent shared key storage and the consequence of not having it (section 3.2). The location is **not decided here**. |
| **Per-edition build outcomes** | [10 — Build and Deployment Requirements](10-build-and-deployment-requirements.md) | Records that a build requiring Windows and Visual Studio *is itself* a cloud-readiness result (section 3.8.4) and cites 10 for the outcomes. It does **not** restate the diagnosis. |

Three further facts are consumed rather than owned here: the security posture of the credentials and connection strings belongs to [09 — Security Assessment](09-security-assessment.md); the performance debt of the per-page aggregate and the tracked-yet-ignored binaries belong to [08 — Technical Debt Register](08-technical-debt-register.md); the EF Core and configuration transitions belong to [05 — ASP.NET Core Migration Approach](05-aspnet-core-migration-approach.md).

---

## 2. Readiness summary

Eight blockers and three favourable findings. Severity is assessed against a managed platform target specifically — an item is Critical where the application would be incorrect or unrunnable as hosted, High where it would be operationally unsound, Medium where it constrains a hosting option rather than the application.

| # | Blocker | Editions | Severity | Target-state replacement |
| --- | --- | --- | --- | --- |
| 3.1 | Cart identity held in in-process session; no shared key material | all three | Critical | Session over a SQL-backed distributed cache, shared data-protection keys, affinity as an interim measure only |
| 3.2 | Data-protection key ring per-instance and ephemeral by default | all three | Critical | Persistent shared key storage — location owned by 06 |
| 3.3 | File-attached database with no cloud analogue | all three, three different engines | Critical | Managed SQL service with encryption in transit, reached without file attachment |
| 3.4 | Windows-authentication connection strings presenting a domain identity | MVC 5, MVC 4 | Critical | Managed identity for data-plane authentication — identity model owned by 06 |
| 3.5 | Configuration and secrets embedded in `Web.config` | all three | High | Platform configuration with secret references, read through a configuration abstraction |
| 3.6 | Plain HTTP, no HSTS, no security response header of any kind | all three | High | HTTPS enforced at the platform edge, HSTS, an explicit security-header set |
| 3.7 | Filesystem path casing mismatch | MVC 5 (demonstrated) | High — gates one hosting option | Repository-wide casing audit, completed **before** a case-sensitive hosting option is viable |
| 3.8 | Statefulness beyond session, and no observability whatsoever | all three | High | Health surface and structured logging — mechanism owned by 06 |
| 4.1 | `HttpContext.Current` appears nowhere | all three | *Favourable* | No static ambient-context coupling to unwind |
| 4.2 | No raw SQL execution anywhere | all three | *Favourable* | No SQL-dialect review to perform |
| 4.3 | No outbound service dependency, no inter-project reference | all three | *Favourable* | Nothing to re-point at a cloud endpoint |

**The shape of the result.** The blockers cluster in infrastructure coupling — session, keys, database attachment, host authentication, transport — and not in application logic. Sections 4.1 to 4.3 are the reason that distinction holds: the code does not reach for a static context, does not write to the filesystem, does not execute raw SQL and does not call anything outbound. What has to change is almost entirely *how the application is composed and hosted*, which is a bounded, largely mechanical body of work. An assessment listing only obstacles would misrepresent that.

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

| Edition | Cart model declaring the constant and `GetCartId` | Constant | `GetCartId` | Four `HttpContextBase.Session` sites | `Controller.Session` site |
| --- | --- | --- | --- | --- | --- |
| MVC 5 | [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs] | `:19` | `:161` | `:163`, `:167`, `:175`, `:179` | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:39] |
| MVC 4 | [src/MVC4/MvcMusicStore/Models/ShoppingCart.cs] | `:19` | `:161` | `:163`, `:167`, `:175`, `:179` | [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:60] |
| MVC 3 | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs] | `:15` | `:166` | `:168`, `:172`, `:180`, `:184` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:22] |

Five sites per edition, fifteen in total:

```bash
# five per edition: four in the cart model, one in the account controller
for e in src/MVC5/MvcMusicStore src/MVC4/MvcMusicStore \
         src/MVC3/MvcMusicStore-Completed/MvcMusicStore; do
  grep -c 'Session\[' $e/Models/ShoppingCart.cs $e/Controllers/AccountController.cs
done
# -> 4 and 1 for each of the three editions
```

**The two key forms matter for what follows.** A signed-in visitor's cart key is their login name — a value that can be recomputed from the authentication ticket on any instance, so it survives anything. An anonymous visitor's cart key is a GUID that exists **only** in the session slot [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:172]: it is never written to a cookie, never returned to the client and never persisted anywhere the next request could recover it. The `Cart` rows keyed by it remain in the database, but the browser-to-GUID link does not, so the rows become unreachable.

#### 3.1.2 Evidence: session storage and key material are both framework defaults

No edition declares a `<sessionState>` element or a `<machineKey>` element in any of its application configuration files. Both are therefore whatever the framework defaults to — in-process session storage, and per-instance key material:

```bash
# 15 application config files; how many declare either element -> 0
git ls-files -- '*.config' '*.Config' | grep -v '/packages/' | grep -v '\.nuget/' \
  | xargs grep -lE '<sessionState|<machineKey' | wc -l
# -> 0     (denominator: the same pipeline without the xargs stage -> 15)
```

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
2. **Shared data-protection keys.** Necessary because distributed session is not sufficient on its own — the session cookie that carries the session id is itself protected by the key ring. Section 3.2 covers this.
3. **Affinity documented as an interim measure, to be switched off once the first two are in place.** Affinity is the reason multiple instances work today, so it must remain until session and keys are shared, and it must then be explicitly disabled — otherwise the platform keeps pinning clients and the resilience the first two controls bought is never actually realized. Leaving it on indefinitely is the failure mode where the work is done and the benefit is not obtained.

A fourth item is a consequence rather than a control: **cutover does not carry anonymous carts across**, because the browser-to-GUID link only ever existed in the old process's memory. That is an approved delta owned by deliverable [05](05-aspnet-core-migration-approach.md), not a defect introduced by the replacement above.

### 3.2 Data-protection keys — "framework-provided" is not "configured"

#### 3.2.1 The distinction

The target framework supplies a data-protection stack without any package reference, which makes it easy to record as solved. It is not. The *stack* is provided; the *key ring* still has to be told where to live, and its default location is local to the instance and does not outlive it.

Today there is no key configuration of any kind in any edition — the same command as section 3.1.2 returns `0` for `<machineKey>`, and no edition declares any alternative key store. Under the current framework, the effect is that each worker generates its own key material.

#### 3.2.2 The consequence

An authentication cookie or anti-forgery token issued by one replica is **rejected by another**, because the second replica cannot decrypt what the first protected. Concretely: a signed-in user's request lands on a different instance and they are silently signed out; a form rendered by one instance is posted to another and the anti-forgery check fails.

The point that matters for sequencing is that **this defeats the distributed session of section 3.1 rather than merely accompanying it**. Moving session into a shared cache makes any instance able to *find* the session, but the session id travels in a protected cookie — so if the key ring is not shared, the second instance cannot read the cookie that identifies the session it can now reach. Control 1 without control 2 does not work.

#### 3.2.3 Target-state replacement

**Persistent, shared key storage**, provisioned and readable by every instance of the application. All instances must resolve one key ring, and the store must survive instance replacement and redeployment.

Deliverable [06](06-azure-hosting-recommendations.md) owns **where the key ring lives**, its protect-at-rest behaviour, its rotation policy, and the isolation between deployment slots or revisions that stops a staging environment decrypting production cookies. Those are hosting decisions and this document does not take them.

One platform-specific escalation belongs here because it changes the requirement's status rather than its content: under a **container-native hosting option**, persistent shared key storage is **non-optional rather than merely advisable**, because replicas there are ephemeral by construction and a key ring on the local filesystem would not survive a revision. Whether that option is selected is a hosting decision owned by [06](06-azure-hosting-recommendations.md); the conditional consequence is recorded here so the dependency is not lost.

### 3.3 A file-attached database has no cloud analogue

#### 3.3.1 Evidence: three editions, three different engines

Every edition reaches its data by attaching a database *file* from the application's own directory, and no managed database service offers an equivalent. This is not a connection-string edit; the storage model is different.

| Edition | Catalog store | Credential store | Attachment mechanism |
| --- | --- | --- | --- |
| MVC 5 | LocalDB `MSSQLLocalDB`, `AttachDbFilename=\|DataDirectory\|\MvcMusicStore.mdf` [src/MVC5/MvcMusicStore/Web.config:13] | a separate file-attached Identity database via `DefaultConnection` [src/MVC5/MvcMusicStore/Web.config:12] | file attachment, both stores |
| MVC 4 | `(LocalDB)\v11.0` with `AttachDbFilename` [src/MVC4/MvcMusicStore/Web.config:19-21] | a separate file-attached SimpleMembership database, `Data Source=(LocalDb)\v11.0` [src/MVC4/MvcMusicStore/Web.config:13] with `AttachDBFilename` [src/MVC4/MvcMusicStore/Web.config:16] | file attachment, both stores, on a **retired instance name** |
| MVC 3 | **SQL Server Compact 4.0** — `Data Source=\|DataDirectory\|MvcMusicStore.sdf` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:57] with `providerName="System.Data.SqlServerCe.4.0"` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:58] | **not declared** — resolved from machine configuration (deliverable 01 §8.3) | an in-process file-based engine |

Three distinct problems, not one repeated three times. MVC 5 attaches files to a LocalDB instance. MVC 4 does the same but names `v11.0`, a SQL Server 2012-era instance its own README contradicts. MVC 3 does not use SQL Server at all: SQL Server Compact is an in-process engine reading a local `.sdf`, with no managed-service counterpart of any kind and no supported provider on the target framework — deliverable [12](12-migration-blockers.md) owns that as a no-successor construct.

`|DataDirectory|` resolves to the application's own `App_Data` folder, which makes the database part of the deployment payload. On a managed platform the application directory is a deployment artifact, not durable storage: it is replaced on deploy and not shared between instances. Two instances would attach two different copies of the same database and diverge.

#### 3.3.2 Evidence: the operational weight this leaves in the repository

Fourteen database binaries are committed, totalling **43,376,640 bytes**:

```bash
git ls-files | grep -icE '\.(mdf|ldf)$'
# -> 14
git ls-files | grep -iE '\.(mdf|ldf)$' | xargs -d '\n' stat -c '%s' \
  | awk '{s+=$1} END {print s}'          # bc is not installed on this host
# -> 43376640
```

| Directory | Files | Bytes |
| --- | --- | --- |
| `src/MVC3/MvcMusicStore-Assets/Data/` | 4 | 18,079,744 |
| `src/MVC4/MvcMusicStore/App_Data/` | 6 | 12,779,520 |
| `src/MVC5/MvcMusicStore/App_Data/` | 4 | 12,517,376 |
| **Total** | **14** | **43,376,640** |

`.gitignore` lists `App_Data/` [.gitignore:32], but an ignore rule cannot untrack a file already added — so these are tracked *despite* being excluded, which is what makes them debt rather than a decision. Deliverable [08](08-technical-debt-register.md) owns that entry and deliverable [09](09-security-assessment.md) owns the exposure, since three of these files are credential stores. The cloud-readiness consequence is narrower and belongs here: every clone, build agent and deployment package carries 41 MiB of database files that the target hosting model has no use for, and a deployment that copies `App_Data` to a managed platform ships data into a directory that will be replaced.

#### 3.3.3 Target-state replacement

**A managed SQL service, reached over the network with encryption in transit, with no file attachment anywhere in the connection string.** The database stops being part of the deployment payload and becomes an external dependency addressed by hostname, so all instances share one authoritative copy and a redeployment cannot replace it.

Two consequences follow and are owned elsewhere: schema and data have to get there, which is the EF Core transition and the data-migration workstream owned by deliverable [05](05-aspnet-core-migration-approach.md); and the instance has to be provisioned with a separation between the identity that applies schema changes and the identity the application runs as, owned by deliverable [06](06-azure-hosting-recommendations.md). MVC 3's provider retirement is owned by deliverable [12](12-migration-blockers.md).

### 3.4 Windows-authentication connection strings present an identity that will not exist

#### 3.4.1 Evidence

Every SQL Server connection string in the two SQL Server editions authenticates with the Windows identity of the worker process:

- MVC 5, catalog store: `Integrated Security=True` [src/MVC5/MvcMusicStore/Web.config:13], and the Identity store likewise [src/MVC5/MvcMusicStore/Web.config:12].
- MVC 4, credential store: `Integrated Security=SSPI` [src/MVC4/MvcMusicStore/Web.config:15]; catalog store: `Integrated Security=True` [src/MVC4/MvcMusicStore/Web.config:21].

MVC 3 declares no credentials at all for its catalog store [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:55-59] — SQL Server Compact is in-process and has no authentication step — and its credential store is resolved from machine configuration rather than the repository, per deliverable 01 §8.3.

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

Relatedly, and often misread as the transform's purpose: **`customErrors` never appears as a live element anywhere in the repository.** There are 24 occurrences across the XDT files and every one is inside a comment block:

```bash
git ls-files -- '*.Debug.config' '*.Release.config' | xargs grep -o 'customErrors' | wc -l
# -> 24
# ...and how many survive comment-stripping, across all 15 application configs -> 0
```

So no edition configures production error behaviour at all. That absence is a runtime decision the port must make explicitly rather than inherit, and deliverable [05](05-aspnet-core-migration-approach.md) owns it together with the error-view replacement.

#### 3.5.3 Target-state replacement

**Platform-supplied configuration with secret references, read through a configuration abstraction rather than from a file in the payload.** Non-secret settings come from the platform's own configuration, secrets are referenced from a secret store rather than held as values, and one built artifact is promotable across environments because nothing environment-specific is inside it. **The administrator credential is removed from source entirely** — not moved to another file, and not carried into the new settings.

Deliverable [05](05-aspnet-core-migration-approach.md) owns the configuration transition and the shape of the settings; deliverable [06](06-azure-hosting-recommendations.md) owns the secret-delivery mechanism.

### 3.6 Transport: plain HTTP, and not one security response header

#### 3.6.1 Evidence

The projects' own web settings serve over plain HTTP with no TLS port configured. MVC 5 enables IIS Express [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:18] with an **empty** `IISExpressSSLPort` element [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:19], and its browse URL is `http://localhost:43524/` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:285]. MVC 4 likewise leaves `IISExpressSSLPort` empty [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:19] with `http://localhost:4321/` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:350]. MVC 3 predates the IIS Express properties entirely — its `IISUrl` element is empty [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:227-228] and it carries only a development-server port [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:225]. No edition configures TLS.

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

A managed platform terminates TLS at its edge and forwards to the application over the internal network. An application that has never been told it is behind such an edge gets two things wrong: it does not redirect insecure requests, and it generates absolute URLs and cookie attributes from the scheme it *sees* rather than the scheme the client used. The result is not a warning — it is mixed-scheme URLs and cookies that fail to travel. So the port must be explicitly configured for edge-terminated TLS; it does not become correct by being hosted somewhere that offers HTTPS.

#### 3.6.3 Target-state replacement

**HTTPS enforced at the platform edge, HSTS enabled, and an explicit security-header set applied by the application.** Insecure requests are redirected rather than served; HSTS instructs browsers not to attempt plaintext; and the header set is chosen and declared explicitly instead of remaining absent. The application is also configured to honour the forwarded scheme so URL generation and cookie attributes reflect the client's actual connection.

Deliverable [06](06-azure-hosting-recommendations.md) owns the hosting-level enforcement and the platform settings that back it. Deliverable [09](09-security-assessment.md) owns the cookie-policy decisions and which of them are hardening versus preservation.

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

**IIS resolves this; a case-sensitive filesystem does not.** On Windows the lookup succeeds because the filesystem is case-insensitive, which is why the defect has never manifested. On a case-sensitive filesystem the file is simply not found. The failure mode is the quiet kind: the application starts, pages return 200, and the site stylesheet is missing — no exception, no failed health check, nothing in a log that would exist anyway (section 3.8.3).

#### 3.7.2 The risk is demonstrated, not theoretical

The repository's own hygiene configuration already contains a mistake of exactly this class:

```bash
git check-ignore -v src/MVC4/MvcMusicStore/.nuget/NuGet.exe; echo "exit=$?"
# -> (no output)
#    exit=1
```

Exit status 1 with no output means **no ignore rule matches the file**. The intended rule is there — `nuget.exe` [.gitignore:28] — but it is spelled in lowercase while the tracked path is `NuGet.exe`, so it does not match and the executable is tracked regardless. Deliverable [02](02-dependency-inventory.md) §5.1 records the client itself and its exact size; deliverable [08](08-technical-debt-register.md) records the tracking as debt. It is cited here for one reason only: it is direct evidence that the casing risk in this repository is real rather than theoretical, and has already produced one live consequence in the repository's own tooling.

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

#### 3.8.2 In-process, per-request work that a shared cache should absorb

The genre menu runs an uncached nested aggregate — a `Sum` over each genre's albums' order-detail quantities, ordered and taken nine at a time [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:46-52] — and because it is a child action rendered from the shared layout it executes on **every page request** (deliverable 01 §5.3).

The readiness consequence is narrow and worth separating from the performance one. This is not instance-affine state, so it does not block scale-out; there is no in-process cache holding it, which is why nothing here breaks when instances multiply. What it does is multiply database round-trips by request volume, so the managed database becomes the scaling bottleneck and the cost driver before the compute tier does. Deliverable [08](08-technical-debt-register.md) owns the performance debt. The readiness note is that the distributed cache introduced for session (section 3.1.4) is the natural home for this aggregate, so the two decisions should be taken together rather than separately.

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

Deliverable [10](10-build-and-deployment-requirements.md) owns the per-edition build outcomes and the toolchain diagnosis, including which checklist items could be satisfied on the verification host and which could not. Those are cited, not restated. The readiness consequence recorded here is the one 10 does not carry: **the pipeline platform is constrained by the framework choice, and that constraint is lifted by the port rather than by the hosting decision** — a cross-platform target framework is what makes a Linux build agent and a Linux container base image available, and no amount of hosting configuration achieves it beforehand.

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
| 2 | Persistent shared data-protection key storage | 3.1, 3.2 | 06 (location, protect-at-rest, rotation, slot isolation) |
| 3 | Session affinity retained as an interim measure, then explicitly disabled | 3.1 | 06 (platform setting), 03 (sequencing) |
| 4 | Managed SQL service, no file attachment, encryption in transit | 3.3 | 05 (EF Core and data migration), 06 (provisioning, DDL separation) |
| 5 | Managed identity for data-plane authentication | 3.4 | 06 (identity model and grants) |
| 6 | Platform configuration with secret references; credential removed from source | 3.5 | 05 (configuration transition), 06 (secret delivery), 09 (credential finding) |
| 7 | HTTPS enforced at the edge, HSTS, explicit security headers, forwarded-scheme handling | 3.6 | 06 (enforcement), 09 (cookie policy) |
| 8 | Repository-wide path-casing audit, as a precondition | 3.7 | 06 (the hosting option it gates) |
| 9 | Health surface reporting dependency readiness | 3.8.3 | 06 (probe configuration) |
| 10 | Structured logging through a logging abstraction | 3.8.3 | 06 (telemetry mechanism) |

**Two ordering constraints are properties of the controls themselves** and are recorded here because they are readiness facts rather than schedule decisions; deliverable [03](03-modernization-roadmap.md) owns the sequencing that acts on them.

- **Control 1 requires control 2 to be in place to deliver any benefit** (section 3.2.2): distributed session is reached through a cookie that the key ring protects, so unshared keys leave a second instance unable to read the session it can now find.
- **Control 3 must be reversed only after 1 and 2 are both live** (section 3.1.4): affinity is what makes the current multi-instance configuration work, and disabling it earlier removes the property that is holding the application together.

---

## 6. Reproducibility appendix

Every command in this document, collected for re-execution. All are read-only: none writes to the working tree, and none contacts the network. Run from the repository root. POSIX forms, executed on this Windows host through the bundled Git-for-Windows `bash`. **`bc` is not installed here**, so byte totals use `awk`.

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

# --- Committed database binaries (§3.3.2) -----------------------------------
git ls-files | grep -icE '\.(mdf|ldf)$'                                                    # -> 14
git ls-files | grep -iE '\.(mdf|ldf)$' | xargs -d '\n' stat -c '%s' \
  | awk '{s+=$1} END {print s}'                                                            # -> 43376640
# per-directory subtotals quoted in the §3.3.2 table
git ls-files | grep -iE '\.(mdf|ldf)$' \
  | while read f; do printf '%s\t%s\n' "$(dirname "$f")" "$(stat -c '%s' "$f")"; done \
  | awk -F'\t' '{c[$1]++; s[$1]+=$2} END {for (d in s) print c[d], s[d], d}' | sort -k3

# --- Configuration transforms (§3.5.2) --------------------------------------
for f in $(git ls-files -- '*.Debug.config' '*.Release.config'); do
  printf '%s total=%s active=%s\n' "$f" "$(grep -c 'xdt:Transform' "$f")" \
    "$(python -c "import re,sys;s=open(sys.argv[1],encoding='utf-8-sig').read();print(re.sub(r'<!--.*?-->','',s,flags=re.S).count('xdt:Transform'))" "$f")"
done                          # -> Release: total=3 active=1 (line 18); Debug: total=2 active=0

git ls-files -- '*.Debug.config' '*.Release.config' | xargs grep -o 'customErrors' | wc -l  # -> 24
# ...none of them live: comment-stripped count across all 15 application configs -> 0

# --- Transport and security headers (§3.6.1) --------------------------------
git ls-files -- '*.config' '*.Config' '*.cs' '*.cshtml' | grep -v '/packages/' \
  | xargs grep -liE 'requireSSL|httpsOnly|RequireHttps|Strict-Transport|customHeaders|X-Frame-Options|X-Content-Type-Options|Content-Security-Policy|Referrer-Policy|Permissions-Policy' \
  | wc -l                                                                                  # -> 0

# --- Path casing (§3.7) -----------------------------------------------------
git ls-files 'src/MVC5/MvcMusicStore/Content/*'          # -> 3 files; Site.css has a capital S
grep -n 'site.css' src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs      # -> :28, lowercase
grep -rn '@Url.Content' src/MVC5/MvcMusicStore/Views/ | wc -l                              # -> 4
git check-ignore -v src/MVC4/MvcMusicStore/.nuget/NuGet.exe; echo "exit=$?"
                                            # -> no output, exit=1: no rule matches NuGet.exe

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
git status --porcelain                       # only new files under docs/modernization/
```

---

## 7. Where each finding is consumed

| Finding | Section | Consumed by |
| --- | --- | --- |
| Session-held cart identity; resilient scale-out foreclosed | 3.1 | 12 (blocker), 05 (session transition), 06 (cache provisioning, affinity setting), 03 (sequencing) |
| Data-protection key ring per-instance by default | 3.2 | 12, 06 (key-ring location and policy) |
| File-attached databases; three engines; 43,376,640 committed bytes | 3.3 | 12, 05 (EF Core, data migration), 06 (provisioning), 08 (tracked binaries), 09 (credential stores) |
| Windows-authentication connection strings | 3.4 | 12, 06 (identity model), 09 (stored-credential posture) |
| Configuration and secrets in the payload; one active XDT transform | 3.5 | 05 (configuration transition, build-vs-runtime split), 06 (secret delivery), 09 (plaintext credential) |
| Plain HTTP; no HSTS; no security header of any kind | 3.6 | 12, 06 (edge enforcement), 09 (cookie policy) |
| Path-casing mismatch; the `.gitignore` corroboration | 3.7 | 06 (the hosting option it gates), 08 (the `NuGet.exe` debt entry) |
| No local filesystem writes | 3.8.1, 4 | 07 (a workstream that is not needed) |
| Uncached per-request aggregate | 3.8.2 | 08 (performance debt), 06 (cache sizing) |
| No logging, tracing, health surface or metrics | 3.8.3 | 12, 06 (telemetry mechanism), 07 (net-new work) |
| Windows-and-Visual-Studio build constraint as a readiness result | 3.8.4 | 10 (build outcomes), 06 (build agent and base image) |
| `HttpContext.Current` absent; no raw SQL; no outbound dependency | 4.1–4.3 | 07 (scoping and effort), 05 (context and query transitions) |

---

*Deliverable 11 of 13. Consumes deliverables [01](01-architecture-overview.md) and [02](02-dependency-inventory.md); feeds deliverable [12](12-migration-blockers.md) alongside 09 and 10. Index and requirement map: [README](README.md). Every count in this document is reproducible by the command stated beside it, and no repository file outside `docs/modernization/` was created, modified or deleted in its production.*
