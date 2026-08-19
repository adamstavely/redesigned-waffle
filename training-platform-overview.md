# SOLUTION OVERVIEW

# A production-grade classroom, on demand

A centralized training platform that configures training spaces inside your deployed systems in minutes, seeds them with versioned simulations, grades what students actually did against the system's own state, and resets for the next class.

*Planning reference, not a commitment · CRUCIBLE training orchestration · GitOps · Kubernetes*

---

## 01 · What this is

This is a self-contained training platform that runs alongside your production estate but never touches it. It gives the training branch one control plane, CRUCIBLE, that configures **training spaces inside the already-deployed applications**: scoped team spaces created through each app's own admin APIs, a simulation seeded from a versioned scenario, identity transformed into a training context, and a session cockpit alongside, all reset and torn down when class ends.

Fidelity is absolute because **training happens in the real, deployed application**, not in a copy. A training space is not an instance of the app; it is a configured context inside it, so there is nothing to drift and nothing to rebuild. Dedicated instances remain the exception, reserved for exercises that touch app-global state such as administration courses.

It keeps two things strictly apart. The **platform team owns the environment as a service**: the cluster, the orchestrator, the reset machinery, the grading engine. The **training branch owns the curriculum**: scenario packs, rubrics, rosters, and courses. The interface between them is versioned artifacts, not tickets, so neither needs the other in the loop for day-to-day operation.

And it grades honestly. Assessment works by **inspecting system state and telemetry**, not by watching screens. An assignment is a set of declarative assertions against the team space: does the expected record exist, is the configuration correct, did the student perform the required sequence. Every score carries per-assertion evidence, and results flow to the LMS, which remains the system of record.

## 02 · What it does

**Ready in minutes.** An instructor picks an app, a scenario, and a roster; the platform configures team spaces and access in the deployed app and loads the simulation. No deployment, just API calls.

**Reset, not cleanup.** Reset re-seeds the simulation or deletes and recreates the team space through the app's own APIs, never a script that tries to undo individual student actions. Same-day turnaround between classes.

**Versioned scenarios.** Seed data, application state, accounts, and injects ship as versioned scenario packs, so every cohort starts from identical, reproducible conditions.

**State-based grading.** Assertions run against the app's own databases, APIs, and audit logs, scoped by team, producing a scored rubric with evidence and instructor override.

**Real identity, training context.** Students authenticate normally against the enterprise IdP. An STS token exchange then applies a training context, issuing a short-lived, audience-locked token that carries cohort, session, and team claims, either as the real user or as a derived training persona for roles they do not hold in real life. No shadow accounts, no separate credential lifecycle.

**No real data, ever.** All data is synthetic, enforced by network isolation rather than policy. The training cluster has no path to production and no mechanism to import its data.

**Instructor console.** Cohort setup, live progress across every student's space, mid-exercise injects, and one-click scenario reset, space rebuild, or teardown.

**LMS integration.** Grades and completion publish over xAPI or LTI, so training records live where the training branch already keeps them.

## 03 · How fast can you reset?

Reset is tiered by scope, not improvised per app. Because the team space and its simulation are both created through the app's own APIs from versioned inputs, every reset is a re-run of known API calls, never an attempt to undo what students did.

**SCENARIO RESET · minutes.** Re-seed the simulation data within the team space through the app's seed hook. The student keeps their space and access; the exercise starts over. The standard verb between sessions.

**SPACE REBUILD · under 15 minutes.** Delete and recreate the team space itself through the app's admin API, then re-seed the simulation. Used after destructive exercises, or whenever a space is suspect.

**COHORT TEARDOWN · scheduled.** Remove every team space, access grant, and training token for the cohort on its end date, automatically. Scheduled auto-teardown prevents sprawl in the shared apps.

**How reset stays cheap.** Training spaces are cattle. Because a space is entirely described by the app's integration contract, the scenario pack version, and the roster, restoring it is a re-run of API calls, not archaeology. Nothing in the platform ever attempts to enumerate and reverse what a student did; it deletes the scoped objects and recreates them. Effort scales with the size of the simulation, not with the creativity of the class.

## 04 · How it works

### A. How a training space is born

An instructor (or the course schedule) requests a training space: app, scenario pack, roster. The orchestrator calls the app's admin API through its integration contract to create team spaces, then runs the seed hook to load the simulation and records the created object IDs as the baseline manifest. Identity is not minted; it is transformed. The student signs in with their real IdP identity, and an STS exchange applies the training context (cohort, session, team, scenario version), issuing a scoped token as the real user or as a derived training persona carrying the real identity in its act claim. The space is then handed to the cohort, ready.

`Instructor request (app · scenario · roster) → Orchestrator → App admin API (create team spaces) → Seed simulation (scenario pack) → Real identity (IdP) → STS exchange → training context → Baseline manifest recorded → Ready`

### B. One contract, many trainings

Each application team publishes one versioned **integration contract**: the admin API operations for creating, resetting, and deleting team spaces, a seed hook for loading simulation data, and a pointer to the app's audit or event feed. From that single artifact the orchestrator can configure any number of training spaces in three modes: **cohort spaces** (one shared team for a class), **individual spaces** (one per student), and **exam spaces** (per student, locked scenario version, submission deadline).

`Integration contract (per app, versioned) → Orchestrator → { Cohort spaces · Individual spaces · Exam spaces }`

### C. How grading works

Student actions land in two places the platform already watches: the application's own audit log and platform telemetry on the activity bus. On demand or at submission, the assertion engine runs the assignment's checks: **state assertions** query the app's databases and APIs, scoped by team, for expected end states, and **event assertions** verify required action sequences from the logs. The engine emits a scored rubric with per-assertion evidence, routes subjective artifacts to an instructor queue, accepts overrides with a required justification, and publishes results to the LMS.

`Student actions → { App audit logs · Platform telemetry } → Assertion engine (state checks + event checks) → Scored rubric + evidence → Instructor review/override → LMS (xAPI / LTI)`

### D. Training vs production, joined by artifacts, never by data

Two estates, one promotion path. Release artifacts, charts, and IaC flow from the production pipeline into training, so the training estate always runs what production runs (N or N-1, with release-candidate pinning for teach-ahead courses). Nothing flows back, and no data crosses in either direction: training egress is default-deny with an allow list limited to the artifact registry, the enterprise IdP, and the STS, and the seed pipeline accepts input only from the scenario registry. The no-real-data rule is a network fact, not a memo.

`Production pipeline → (artifacts only, one way) → Training estate`
`Production data ✕ never crosses · Training spaces ✕ never reach production`

### E. Where it runs

Everything runs on the dedicated training Kubernetes cluster, mirroring the production topology at reduced size: same ingress pattern, same mesh configuration, same observability stack, same secrets management. The cluster splits into two zones: a **shared core zone** holding CRUCIBLE's services (orchestrator, scenario registry, assertion engine, STS context transform, telemetry, mocks), and an **application zone** hosting the deployed training apps themselves, shared web applications partitioned by team space and training context rather than by copies.

`Training cluster (isolated) = Shared core zone [orchestrator · scenario registry · assertion engine · STS transform · telemetry · service mocks] + Application zone [deployed training apps · team spaces created and destroyed on demand]`

## 05 · Built on

Components added by the roadmap phases, each doing one job.

**ROADMAP ADDITIONS**

- **Assertion engine** · Phase 2 · grading. Declarative state and event assertions, weighted rubrics, evidence capture, instructor override with justification.
- **Student portal** · Phase 2 · access. Credential delivery, assignment view, submission, and results, per student, per cohort.
- **LMS bridge** · Phase 2 · records. xAPI / LTI publisher so grades and completions land in the training branch's system of record.
- **Space snapshot service** · Phase 2 · multi-day. Export and restore of a team space's scoped state for multi-day courses, where the app's APIs support it.
- **Synthetic data toolkit** · Phase 3 · authoring. Generators for common entity types plus pack-authoring tooling the training branch uses without platform help.
- **Self-service onboarding** · Phase 3 · scale. App teams publish and maintain their own integration contracts through a documented spec and conformance checks.
- **Cohort automation** · Phase 3 · operations. Scheduled provisioning, end-date auto-destroy, and capacity management across the calendar.
- **RC cohorts** · Phase 4 · teach-ahead. Point training spaces at release-candidate deployments so cohorts learn upcoming releases and double as structured pre-rollout usability tests.
- **Classroom packs** · Phase 4 · portability. Export an integration contract, scenario, and course configuration as one portable bundle for disconnected or remote delivery.

## 06 · Adopt before build

The portal and session layer of this platform is a solved problem in open source. Adopting it shrinks the custom build to the three things only we can write: training profiles, seed and reset hooks, and the assertion checkers that grade against our own applications.

**SESSION AND PORTAL LAYER**

- **Educates** · recommended base. Apache 2.0 platform for hands-on workshop environments: each student gets an isolated session with step-by-step instructions, integrated terminals, and an embedded editor, fronted by a training portal that handles cohorts, capacity, access codes, session expiry, and orphaned-session cleanup. Originally built for Kubernetes training but teaches any topic, supports supervised or self-paced modes and custom branding, and is explicitly designed for self-hosting, including air-gapped deployment. Now community-stewarded after leaving VMware/Broadcom, so maintainer bus-factor goes on the risk register.
- **Educates, shared-app pattern** · centrally hosted systems. Sessions do not have to contain the app. For a shared multi-tenant application, the app deploys once (environment-level resources or entirely outside Educates), a per-session setup script calls the app's admin API to create a team keyed to the session name, credentials and parameters inject via session data variables, and the dashboard embeds the app as a tab (the app's frame-ancestors policy must allow it, or a launch-in-new-tab button dodges the issue). Teardown deletes the team. Reset becomes an API call, not a snapshot restore. The constraint: exercises must be expressible within a team's scope; anything app-global needs a dedicated instance, the exception path.
- **Hobbyfarm** · VM-first alternative. Rancher's browser-based training platform, provisioning full virtual machines and streaming desktops through Guacamole. Used by Rancher Government for public-sector Kubernetes training. The right base if courses need OS-level or desktop-application access rather than web UIs and terminals.
- **Apache Guacamole** · desktop access. Clientless remote desktop gateway (RDP, VNC, SSH) for embedding access to thick-client or legacy tools inside a browser session, standalone or underneath Hobbyfarm.

**ASSESSMENT AND RECORDS**

- **CTFd** · lightweight scoring. Challenges, flags, hints, scoreboards, teams, and a plugin API. An exercise step emits a flag only when our checker passes, which gives scoring, progress, and leaderboards without building any of it.
- **PrairieLearn** · serious autograding. Containerized external graders, question banks and randomization, exam mode, and LTI integration. The heavier option when assessments must be defensible and auditable rather than gamified.
- **CyberRangeCZ (KYPO)** · pattern reference. Masaryk University's open-source cyber range with linear training phases, scoring, and instructor monitoring. Adopt directly for security-flavored training, or mine it for assessment-design patterns.
- **Moodle / Open edX** · LMS of record. If the training branch lacks an LMS, either provides enrollment, records, and the LTI/xAPI endpoint our results publish to. If an LMS exists, it stays; we only integrate.

**WHAT REMAINS CUSTOM**

Three artifacts, all thin: the integration contract and its conformance checks, the seed and reset hooks each app team implements against that contract, and the assertion checkers that query our applications' APIs, databases, and audit logs by team or session ID. Everything else, portals, session lifecycle, scoring, records, is adopted. The practical effect on the plan: Phase 1 becomes an integration exercise, Educates as the cockpit and portal over the apps' own admin APIs, rather than a control-plane build, and Phase 2's grading engine becomes checkers plus CTFd or PrairieLearn rather than a bespoke service.

## 07 · Roadmap

Phases are grouped by dependency, not appeal. The foundation comes first because grading, self-service, and portability are all contracts and views over it.

**Now · delivered design · Control plane and estate.** *In baseline.* The CRUCIBLE design: orchestrator, integration contracts, scenario packs, tiered reset, the STS training-context transform, and the artifact-only boundary with production.
Training space orchestration · Integration contracts · Scenario packs · Tiered reset · Training context transform

**Phase 1 · Foundation · ~90 days.** Two or three flagship apps onboarded with integration contracts, cohort training spaces configured through their admin APIs, scenario reset working end to end, one scenario pack per app, and minimal instructor operations. Exit test: one real course taught end to end with a same-day reset between sessions.
First integration contracts · Cohort spaces · Scenario reset · First scenario packs · Instructor operations

**Phase 2 · Assessment · next 90 days.** The grading engine with state and event assertions, the student portal, individual and exam spaces, and the LMS bridge. Exit test: a graded course where instructors touch grading only for overrides and subjective artifacts.
Assertion engine · Student portal · Individual spaces · Exam spaces · xAPI / LTI

**Phase 3 · Scale · following 90 days.** Self-service onboarding for app teams, scenario authoring tooling with synthetic data generators, scheduled cohort automation with auto-destroy, and capacity management. Exit test: the training branch runs cohorts with no platform team in the loop.
Self-service onboarding · Authoring toolkit · Cohort automation · Auto-destroy · Capacity management

**Phase 4 · Teach-ahead and portability.** Release-candidate cohorts that double as pre-rollout usability tests, and classroom packs for disconnected delivery. Gated on Phases 1 through 3 being boring.
RC pinning · Pre-rollout feedback loop · Portable classroom packs

## 08 · What it costs to run

Planning-grade figures for a dedicated training cluster. The baseline is the shared core; each phase adds an increment, and cohort load adds a small per-cohort marginal cost on top, mostly capacity headroom in the shared apps and simulation data. Two postures: idle (no cohorts running) and loaded (a representative four concurrent cohorts).

| Build stage | Adds | Idle total | Loaded total |
|---|---|---|---|
| Core estate (baseline) | · | ~$2,400 | ~$3,200 |
| + Phase 1 · foundation | +$400 | ~$2,800 | ~$3,700 |
| + Phase 2 · assessment | +$500 | ~$3,300 | ~$4,300 |
| + Phase 3 · scale | +$300 | ~$3,600 | ~$4,700 |
| + Phase 4 · portability | +$200–400 | ~$3,800–4,000 | ~$4,900–5,200 |

The structural point matters more than the numbers: because a cohort adds capacity headroom and simulation data in already-deployed apps rather than new deployments, the **marginal cost of a cohort is small**, roughly $150 to $300 per month, and scheduled teardown keeps it from accumulating. Spend follows the training calendar. Dedicated instances for app-global exercises are the exception that carries real deployment cost, which is one more reason to keep them rare. Figures cover compute, storage, and cluster fees; they exclude LMS licensing (already held) and the curriculum development effort in the training branch, which on a program like this is usually the larger investment. Treat these as planning estimates, correct in shape, not quotes.

---

*Solution overview · CRUCIBLE training orchestration · GitOps on Kubernetes · planning reference, not a commitment*
