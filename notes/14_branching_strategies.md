## Strategies for Branching

Before choosing a branching strategy, it helps to decide what you’re optimizing for: speed, safety, or simplicity. Different teams and projects lean different ways—startups with small codebases won’t work the same as larger, multi-repo setups. This overview lays out the options and how to adapt them to your workflow.

### Trunk-Based Development

In trunk-based development, **a single long-lived branch—usually `main`, `master`, or `trunk`—is the integration point for all code**. Small, short-lived feature (topic) branches are fine, but they *must* merge back to trunk quickly—ideally within hours, and rarely later than a day or two.

Every commit to trunk triggers the CI pipeline: it builds, runs the full automated test suite, performs static analysis, and—if green—deploys to a staging/integration environment. After acceptance tests pass, the **same artifact** is promoted toward production unchanged.

This approach depends on **continuous integration**, **strong automated tests**, and **fast feedback loops** so trunk stays releasable.

```
Main Branch (Trunk)
   |
   |——> Continuous Integration (CI)
   |         |
   |         v
   |  Automated Testing
   |
   |——> Short-Lived Feature Branches (optional)
   |         |
   |         v
   |  Feature Development
   |
   |——> Merge Back to Trunk (at least daily)
   |
   |——> Staging → Production Release
```

**How are big breaking changes introduced?**

Use incremental, backward-compatible steps: feature flags/toggles, *branch by abstraction*, and “expand/contract” migrations (e.g., add new paths first, dual-write, then remove the old). If a short branch is unavoidable, keep it tiny, merge daily, and hide behavior behind flags.

**Is every commit here a transaction?**
Treat each commit as **atomic and releasable**: it should build, pass tests, and leave user-visible behavior unchanged by default (flags off). Commits trigger CI and often deploy to staging, but **releases to users are controlled** by flag flips or artifact promotion—not by every commit.

#### Advantages

* A single trunk minimizes merge and release overhead; multiple long-lived branches create bottlenecks. For example, a startup can ship daily without heavy integration work.
* Small, frequent releases reduce risk versus bundling big updates; an e-commerce site can roll out checkout tweaks weekly instead of quarterly.
* Early integration surfaces conflicts quickly; delaying merges makes them harder and costlier. A game studio avoids last-minute crunch by merging daily.
* Keeping trunk releasable enables continuous deployability; otherwise teams get stuck stabilizing. A SaaS provider can hotfix anytime without waiting for a hardening sprint.

#### Disadvantages

* Incomplete work needs **feature flags** to ship safely; without them, unfinished functionality can leak. A payments app hides a new API behind a toggle until testing is done.
* Large or architectural changes require **branch by abstraction** or expand/contract techniques; skipping them risks destabilizing trunk. A cloud service splits a database migration into small, deployable steps.
* Success hinges on **disciplined automated testing**; neglect leads to fragile releases. A healthcare platform runs full regression suites on every commit.
* Build stability suffers if **merge frequency** drops; regular integration prevents long disruptions. A large open-source project enforces daily merges to avoid spikes in broken builds.

#### When to Use

* Teams **practicing daily CI** benefit most; merging weekly or less blunts the gains. A fintech team commits multiple times per day to maintain flow.
* Projects that can be **tested in independent slices** fit well; tightly coupled systems struggle. A microservices platform evolves each service without blocking others.
* Organizations aiming for **continuous delivery** gain speed and safe rollback; long, gated release trains lose much of the benefit. A streaming provider ships fast with instant rollback if needed.
* Heavy regulatory gates or **big-bang releases** reduce suitability; such environments often keep long-running release branches. A defense contractor bound by compliance stages remains on release branches.

#### Critique

* Critics say this model only works well when a team has excellent automated tests and very reliable CI.
* Without that, frequent merges to the main branch feel risky.
* Feature flags (used to hide unfinished work) add complexity and can accidentally expose half-built features.
* Big, sweeping changes are hard to slice into tiny safe steps, so they can feel awkward or slow.
* If the build pipeline is flaky, everyone gets blocked.
* Some also find lots of small PRs reduce review depth and make the history noisy.

### Release Branches

In the **release-branching** workflow, each upcoming version is stabilised on its own branch while everyday feature work continues elsewhere. A typical setup includes:

* `main` (or `master`) – holds production-ready code and immutable version tags (e.g., `v2.3.0`).
* `develop` – the integration branch for day-to-day feature work.
* *Release branch* – cut from `develop` when you declare the next version feature-complete; only bug fixes, documentation tweaks, and release housekeeping (version bumps, build scripts, localisation strings, etc.) are allowed.
* *Hotfix branch* – cut directly from `main` to address critical production issues; merge back into both `main` *and* `develop` (or the active release branch) to prevent regressions.

```
main
 │
 ├──▶ develop
 │       │
 │       ├──▶ Feature branches (feat/XYZ)
 │       │        │
 │       │        ▼
 │       │  Merge back to develop
 │       │
 │       └──▶ Release-X.Y branch
 │                │
 │                ├──▶ Bug fixes / final hardening
 │                │
 │                └──▶ (if urgent) Hotfix branch
 │                         │
 │                         ▼
 │                     Merge to Release-X.Y
 │
 ├──▶ Merge Release-X.Y → main & tag vX.Y.0
 │
 ▼
Production deployment
```

Once the release branch is declared stable:

1. *Merge into `main`* and create an annotated, immutable tag (`vX.Y.0`) for traceability.
2. *Merge (or rebase) back into `develop`* so ongoing work includes all post-freeze fixes.
3. *Delete (or archive) the release branch* to avoid clutter.

#### Advantages

* *Parallel version support* lets you stabilise one release while new features move forward elsewhere (e.g., an ERP vendor patches `v3.2` while building `v3.3`).
* *No feature flags on the release branch*—only completed work lands there, reducing runtime complexity (a mobile team ships toggles-free releases).
* *Scoped hotfixes* target just the affected version, avoiding unintended side effects (a cloud provider patches `v2.x` without touching `v3.x`).
* *Predictable milestones* give QA and stakeholders a clear test scope (a quarterly on-prem release locks features so validation is focused).

#### Disadvantages

* *Slower cadence*—features may wait for the next window, whereas CI/CD on trunk moves faster (app-store cycles can delay delivery).
* *Merge overhead*—fixes must be propagated across branches; a single-branch model avoids duplication (teams on `v1.x` and `v2.x` copy fixes twice).
* *Branch sprawl*—more active branches complicate tooling and mental models; small teams often prefer a single trunk.
* *Divergence risk*—long-lived branches drift and make merges painful; frequent integration keeps differences small.

#### When to Use

* Products with *formal release schedules* (e.g., quarterly enterprise software).
* Situations with *multiple supported versions* in the wild (supporting `v1.x` for legacy and `v2.x` for new customers).
* Environments with *compliance/audit gates* where clearly scoped releases map to documentation (e.g., medical devices).
* Less suitable for teams pursuing *continuous delivery*, where frequent releases and feature toggles already work well.

#### Not the same as Git Flow

*Git Flow* is a full branching methodology. *Release branches* are a single ingredient—often used within Git Flow, but also viable on their own.

#### Critique

* Extra ceremony and slower pace can frustrate teams.
* “Code freeze” on the release branch pauses new work.
* Fixes often have to be merged twice (into the release and back into develop), which is tedious and error-prone.
* Long-lived release branches drift from the main line, causing painful merge conflicts later.
* The workflow encourages big “all-at-once” releases instead of small, frequent ones.
* Tracking changes across many branches makes automation and debugging harder.

### Feature Branches

In a **feature-branch** workflow, each new capability, enhancement, or bug fix lives on its *own* branch cut from a stable integration branch (`main` or, more often, `develop`). Work proceeds in isolation until the change set is *complete* and the CI pipeline is green. The branch is then merged back via a pull/merge request after code review and automated checks. The resulting commits flow to production in the next release train—or immediately, if you practise continuous delivery.

Typical naming conventions: `feat/<ticket-id>-<slug>` for new functionality and `fix/<ticket-id>` for defects. This keeps context clear for both tooling and humans.

```
main
 │
 ├──▶ develop   (optional integration branch)
 │       │
 │       ├──▶ feat/ABC-123-cool-feature
 │       │        │
 │       │        ▼
 │       │   Commit → test → review
 │       │        │
 │       └──▶─────┘  Merge to develop
 │
 └──▶ (CI) → Release preparation → Production
```

#### Advantages

* *Isolation* keeps incomplete work off trunk, reducing broken builds; e.g., a fintech team develops a new payment flow separately until it’s production-ready.
* *Parallel development* lets multiple efforts progress without constant conflicts; a web platform team can add search while redesigning the UI.
* *Focused reviews* on small, scoped PRs improve signal and accountability; an open-source project reviews only the new API endpoint in a single branch.

#### Disadvantages

* *Integration drift* grows on long-lived branches; frequent rebases/merges keep code close to trunk. A mobile team faces multi-day conflicts after a month-long branch.
* *Delayed feedback* hides interaction bugs until merge; early, regular syncs surface issues sooner. A logistics platform discovers a shipments/invoicing bug at final merge.
* *Discipline required*—branches should be short-lived; neglect leads to stale, hard-to-merge work. A game studio rebases daily and closes branches within a week.

#### When to Use

* Teams that value *explicit review gates* and an auditable trail of changes; a government IT project treats each MR as a compliance artefact.
* Projects delivering *independent features* in days or weeks; a SaaS product ships dashboard filters via short-lived branches.
* Organisations not yet ready for *trunk-based development* can reduce risk with scoped branches; teams already practising trunk-based merges may not need them.

Avoid very long-running feature branches. If a change is large, break it into incremental slices or use *branch-by-abstraction* so you can merge to trunk early and often.

#### Critique

The main complaints are about delay and drift. When branches stay open for weeks, they fall behind the main codebase and merge conflicts pile up. Pull requests can grow large, slowing reviews and hiding problems until late in the process. Rebased/updated branches burn time, CI runs are duplicated for many branches, and the PR gate can replace early, direct collaboration with a slower “throw it over the wall” review cycle. Stale branches and lingering PRs sap momentum.

### Forking

In a **fork-based** workflow, contributors work on their *own* copy of the repository—a *fork*—instead of pushing branches to the canonical project. Changes flow back to the *upstream* repository via pull requests (PRs; “merge requests” on some platforms), where maintainers review, discuss, and merge.

Typical fork setup:

* *Upstream* – the original, authoritative repository. In a contributor’s clone, this is usually added as the `upstream` remote (while `origin` points to the fork).
* *Fork* – a full clone under the contributor’s account or an internal team namespace.
* *Feature/topic branches* – created in the fork for each unit of work.
* *Pull request (PR)* – proposes the branch for inclusion upstream; CI runs, reviewers comment, and maintainers merge (often with **squash** or **rebase-and-merge**) to keep history tidy.

```
Upstream repo
   │
   ├──▶ main (production)
   │       │
   │       └──▶ (optional) topic branches
   │
   └──▶ Forked repo (contributor / team)
           │
           ├──▶ main (kept in sync with upstream)
           │
           └──▶ feat/XYZ-topic
                     │
                     ▼
                Development & local CI
                     │
                     ▼
                 Pull request ▲
                               │
                               ▼
                  Review → Merge → Upstream main
```

#### Advantages

* Enforcing a *safe contribution model* by requiring PRs from forks prevents direct changes to upstream; skipping this risks accidents like force-pushes. A popular JavaScript library, for example, avoids exposing write access to casual contributors.
* Operating with *low coupling* lets each fork run its own CI and manage work independently; sharing a single namespace often causes collisions. A research consortium can experiment per university before proposing upstream changes.
* Scaling to *large contributor bases* is feasible when maintainers control merges centrally; direct-commit models don’t scale under volume. Global projects can accept patches from thousands without granting upstream write access.

#### Disadvantages

* Poor *sync hygiene* causes forks to drift; regular fetch/rebase keeps contributors aligned with upstream. A contributor to an ML framework may face hours of conflict resolution after months without syncing.
* *Review bottlenecks* slow progress when maintainers face heavy PR queues; smaller, curated contributor pools reduce wait times. A blockchain project can see week-long delays before patches are merged.
* Duplicated *CI runs* across forks inflate costs; consolidated pipelines minimise waste. An enterprise open-sourcing an SDK may see cloud CI bills climb as hundreds of forks trigger builds.

#### When to Use

* Open-source projects that rely on *external contributions* benefit most; closed teams often gain little from the added overhead. A web framework community depends on forked PRs to manage outside patches.
* Internal platforms with *strict permission boundaries* gain isolation through forks; permissive repos may prefer simpler branch-based models. A multinational lets subsidiaries adapt code safely in their own forks.
* *Security-sensitive environments* can enforce signed PRs and audit trails through forks; looser workflows risk unverified changes. A defence contractor requires every patch to arrive via a forked PR.

Keep forks fresh by adding the upstream remote (`git remote add upstream <url>`), then regularly syncing: `git fetch upstream` followed by `git rebase upstream/main` (or `git pull --rebase upstream main`). Stale forks are the main pain point here; proactive syncing and small, focused PRs keep the workflow smooth.

#### Critique

Contributors often struggle to keep their fork in sync with the upstream project; long gaps mean ugly conflicts later. CI can be weaker on forks (secrets and internal checks may not run), so maintainers must re-run jobs, adding friction. A small maintainer team can’t review a huge PR queue quickly, so contributions sit for days or weeks. Discussion and context spread across many forks and PRs, which makes triage and coordination harder. It’s secure, but high-friction for frequent collaborators.

### Git Flow

**Git Flow** is a prescriptive branching model, proposed by Vincent Driessen, that assigns *named roles* to branches with clear rules for when to create, merge, and retire them.
It suits projects that ship *versioned* releases on a predictable cadence and need to patch production urgently while regular feature work continues.

* `main` – always reflects what’s running in production and is tagged (`vX.Y.Z`) at each release.
* `develop` – the permanent integration branch for day-to-day development.
* `feature/*` – short-lived branches cut from `develop` for each new capability or bug fix; merge back into `develop` when complete.
* `release/*` – cut from `develop` once the next version is feature-complete; only stabilisation changes (bug fixes, docs, build metadata) are allowed; merges into both `main` and `develop`.
* `hotfix/*` – cut directly from `main` to address urgent production defects; merges into both `main` *and* `develop` (and any active `release/*` branch).

```
main ──┬─────────────▶ (tag vX.Y.0) ──┐
       │                              │
       │◀─ merge hotfix/* ──────▶ (tag vX.Y.1)
       │                              │
develop│◀───────────── merge release/*┘
  │
  ├──▶ feature/ABC-123
  │        │
  │        └──▶ merge → develop
  │
  └──▶ release/X.Y
           │
           ├──▶ final bug fixes
           └──▶ merge → main  &  merge → develop
```

#### Advantages

* *Role clarity* gives each branch a distinct purpose; without it, responsibilities blur and contributors get confused. For example, a banking platform keeps ongoing work on `develop` while urgent fixes go to `hotfix/*`.
* *Parallel work streams* allow teams to develop features, stabilise a release, and patch production simultaneously; a mobile app team can ready `v2.1` on `release/*` while starting `v2.2` features on `develop`.
* *Release hardening* phases surface late bugs without halting new work; an on-prem ERP vendor stabilises `v5.4` on `release/*` while roadmap features continue on `develop`.

#### Disadvantages

* Five branch types add *process overhead*; simpler models need less training and coordination. A midsize SaaS finds Git Flow slower to onboard than trunk-based workflows.
* *Long-lived divergence* makes merges painful; keeping branches short reduces integration risk. A telecom vendor struggles when `release/*` drifts weeks from `develop`.
* Typically a *slower cadence* than trunk-based or short-lived branching; a startup delays customer-visible improvements until the next scheduled release.

#### When to Use

* Products with *formal release schedules* benefit, while continuously deployed services may find it heavy. A quarterly enterprise desktop product fits well.
* Organisations with *separate Dev, QA, and Ops roles* gain from explicit branch stages; cross-functional teams may prefer lighter models. A regulated insurer requires QA sign-off before merging into `release/*`.
* Maintaining *multiple supported versions* makes hotfix branches valuable; single-version products rarely need them. A medical software provider patches `v3.x` in production while continuing work on `v4.x`.

If you aim for *continuous delivery* or deploy many times a day, Git Flow will likely feel heavyweight; consider trunk-based development or a simple feature-branch workflow instead.

#### Critique

* Considered *heavy and dated* for modern CI/CD; detractors argue it slows down teams used to rapid iteration.
* The *many branch types* add process and slow onboarding; new team members face a steeper learning curve.
* The permanent `develop` branch can feel like an unnecessary extra hop, complicating the path to production.
* `release/*` and `hotfix/*` branches add merge overhead and increase the risk of divergence; teams report painful merges when branches drift apart.
* Teams often experience *slower feedback* and *slower releases* compared to trunk-based or short-lived feature branch workflows.
* Histories can become cluttered with coordination merges, making it harder to read and understand the commit log.

### Environment Branching

In an **environment-branching** workflow, each *runtime environment*—development, QA, staging, production—has its own long-lived branch. Code moves “up the ladder” by merging (or cherry-picking) from one environment branch to the next, mirroring how build artefacts and database changes are promoted.

Typical branch mapping:

* `develop` – the developers’ sandbox; unstable experiments are expected.
* `qa` (or `test`) – mirrors the formal testing environment; only code that has passed review and automated checks is promoted here.
* `staging` – a pre-production clone of live; final acceptance, performance, and compliance testing happen here.
* `main` (or `production`) – tracks what is currently in production; immutable tags (`vX.Y.Z`) mark each deployment.

```
develop ──┬──────────▶ (dev environment)
          │
          └──▶ qa ────┬────────▶ (QA / test environment)
                      │
                      └──▶ staging ───▶ (staging environment)
                                          │
                                          ▼
                                      main (prod) ──▶ (production)
```

Promotion flow:

1. Finish work on `develop` → merge to `qa`.
2. QA passes → fast-forward or merge `qa` → `staging`.
3. Final sign-off → merge `staging` → `main` and deploy.
   *Hotfixes* start on `main`, then cascade back to `staging`, `qa`, and `develop` to keep branches aligned.

#### Advantages

* Maintaining *environment visibility* means each branch maps to a deployed environment, whereas skipping this blurs traceability; for example, a bank’s `qa` branch reflects exactly what testers are validating.
* Using *paced promotion* lets changes pause in QA or staging for deeper checks, whereas merging straight to production risks unvetted code; a medical-device firm runs extended regulatory tests in `staging` before promoting.
* Aligning with *regulatory expectations* provides auditors a clear branch-to-environment link, whereas missing this mapping complicates evidence; an aviation vendor can point regulators to the precise branch under test.

#### Disadvantages

* Orchestrating *merges across environments* adds complexity, whereas lighter models need fewer moves; a finance team spends hours cherry-picking fixes through `qa`, `staging`, and `prod`.
* Allowing *branch divergence* breeds “works-on-my-env” bugs, whereas disciplined syncing prevents drift; a telecom sees a prod incident when a `staging` patch never reaches `develop`.
* *Slower feedback loops* postpone integration issues, whereas trunk-based flows surface them early; a logistics platform discovers conflicting features only after days sitting in `qa`.
* Supporting *multiple environment branches* strains CI/CD, whereas single-branch pipelines stay lean; a legacy ERP vendor pays for duplicated builds across several environment branches.

#### When to Use

* Organisations with *rigid stage gates* (e.g., FDA/aviation) benefit, whereas less-regulated sectors may not need the overhead; a medical records system must pass certified environments.
* *Separate Dev, QA, and Ops teams* align well with environment ownership, whereas integrated product teams may prefer trunk-based flows; a bank assigns `qa`, `staging`, and `prod` to different groups.
* *Legacy, heavyweight deployments* justify extra staging steps, whereas modern cloud-native platforms may prefer faster rollback strategies; an insurer running mainframe code relies on staging time to mitigate rollback risk.

If rapid, continuous delivery is the goal, consider slimming the number of environment branches—or switch to release branches or trunk-based development plus feature flags—to keep promotion fast while retaining control.

#### Critique

Common complaints say this model uses Git branches for the wrong job: branches end up representing *environments* (dev/QA/staging/prod) instead of *changes to the code*, which leads to drift between branches, lots of cherry-picking, and hard-to-trace deployments. Best practice is to build **once** and promote the **same artifact** through each environment, changing only configuration; environment branches usually break that rule, so “staging” and “prod” run different commits and bugs slip through. Configuration should live outside the code (env vars, secrets, config stores), not as separate code branches, which is a core 12-factor principle. Teams also point out that per-environment branches multiply CI pipelines and make rollbacks and audits harder because you can’t point to one SHA that moved through the pipeline unchanged. These critiques show up across CD/GitOps guidance and are a big reason many call “branch-per-environment” an anti-pattern.
