# ADR-0010: Crucible Configures Training Spaces Inside Deployed Applications via Versioned Integration Contracts

| Field | Value |
|---|---|
| **Status** | Proposed |
| **Date** | 2026-08-18 |
| **Decision Owner** | Technical Director |
| **Consulted** | Principal Engineers, Platform Tech Leads, Training Branch Lead |
| **Informed** | All engineering teams, Training Branch |
| **Related ADRs** | ADR-0002 (C4 with Mermaid), ADR-0003 (DDD for capability platforms), ADR-0006 (Platforma IDP), ADR-0008 (enterprise OAuth/STS/OBO) |

## Context and Problem Statement

The training branch needs a repeatable way to run hands-on training on our internal applications: configure student access, load exercise simulations, reset between classes, and grade what students actually did. Today there is no dedicated mechanism. Training happens ad hoc in whatever environment is available, cannot be reset reliably, and cannot be graded against system state.

The internal applications students must learn are centrally hosted, multi-tenant web applications that are already deployed. The architectural question is what the unit of training is: a provisioned copy of each application per cohort, or a configured context inside the deployed application itself.

If no decision is made, each course improvises. Training either touches shared environments with no isolation and no reset, or individual teams hand-build training copies of their apps that drift from the deployed reality and teach a system that does not exist. Grading remains manual observation, and every course carries its own unrepeatable setup cost.

## Decision Drivers

- Fidelity: students must train on the system they will actually use, at the version actually deployed
- Reset must be fast, reliable, and safe to run between same-day sessions, and must never depend on undoing individual student actions
- Grading must be defensible: derived from system state and audit evidence, reproducible from identical starting conditions
- Onboarding cost must sit with application teams, who hold the app knowledge, through a versioned artifact rather than tickets
- Identity must remain the enterprise identity: real IdP authentication, transformed rather than duplicated, consistent with ADR-0008
- The platform team must not become an operator of per-course infrastructure; marginal cost per cohort must be small
- Build effort should concentrate on what only we can write; commodity portal and scoring layers should be adopted, not built

## Options Considered

### Option 1: Provision dedicated application instances per cohort or per student

Stamp out isolated copies of each application from production charts and artifacts for every cohort, with snapshot-based reset (Velero plus database snapshots) and a separate training identity realm.

- **Pros:** Structural isolation; app-global exercises possible everywhere; reset is a restore regardless of app cooperation; no app-side API requirements
- **Cons:** Every course carries a deployment; per-cohort marginal cost is dominated by app infrastructure; a parallel identity realm creates a shadow account lifecycle; parity with the deployed apps must be actively maintained; app teams must produce and maintain deployable training profiles
- **Risks:** Medium. Drift between training instances and deployed reality; environment sprawl; the platform team becomes a training-infrastructure operator

### Option 2: Configure training spaces inside the deployed applications via versioned integration contracts (chosen)

The unit of training is a training space: team spaces created through each application's own admin API, a simulation seeded from a versioned scenario pack, identity transformed into a training context by the enterprise STS (RFC 8693 token exchange per ADR-0008, with the real identity carried in the act claim when a derived persona is used), and a session cockpit alongside. Each application team publishes one versioned integration contract: space create/delete/access operations, a seed hook, and a pointer to its audit or event feed. Reset is a re-run of known API calls (scenario reset, space rebuild, cohort teardown), never an undo. Adopted open source carries the portal and scoring layers (Educates; CTFd or PrairieLearn); custom code is limited to the orchestrator, the contracts, and the assertion checkers. Dedicated instances remain a governed exception for exercises that touch app-global state.

- **Pros:** Absolute fidelity, since training happens in the deployed application; small per-cohort marginal cost (capacity headroom and simulation data); no shadow identity; onboarding cost sits with app teams as a versioned contract; grading keys on the same team and session identifiers the token carries; reset is deterministic API replay
- **Cons:** Requires each application to expose team-scoped admin APIs and honor training-audience tokens; exercises are limited to what a team scope can express; the training/production boundary becomes token validation rather than structural separation, so it must be conformance-tested per app; noisy-neighbor risk in shared apps during load-heavy exercises
- **Risks:** Medium-low. A replayable training token against production is the principal failure mode; mitigated by distinct issuer and audience, per-app rejection tests in contract conformance, and short TTLs bound to session lifetime

### Option 3: Continue ad hoc, manually built training environments

Each course owner assembles whatever environment they can, documented in runbooks.

- **Pros:** No platform investment; full flexibility per course
- **Cons:** No reset, no reproducibility, no defensible grading, unbounded per-course setup cost, drift guaranteed
- **Risks:** High. This is the current state the decision exists to correct

### Option 4: Commercial hands-on training platform (Instruqt, Appsembler, or similar)

License a hosted training platform and integrate our applications into its lab model.

- **Pros:** Mature portal and lab UX immediately; vendor carries the session lifecycle
- **Cons:** Built around provisioned sandboxes rather than configured contexts in deployed apps, so it reintroduces Option 1's model at licensed cost; data residency and connectivity constraints in our environment; grading against our applications' state remains custom work regardless
- **Risks:** Medium. Vendor dependency for a capability whose hard parts (contracts, checkers) we must build either way

## Decision Outcome

Option 2. Crucible configures training spaces inside the already-deployed applications through versioned integration contracts, with identity transformed via the enterprise STS per ADR-0008, reset implemented as deterministic API replay, grading asserted against team-scoped application state and audit evidence, and adopted open source (Educates for the portal and cockpit; CTFd or PrairieLearn for scoring) carrying the commodity layers. Dedicated instances are the governed exception path for app-global exercises and require AERB awareness at course design time.

## Consequences

### Positive

- Training fidelity is inherent rather than maintained; there is no copy to drift
- Marginal cost per cohort is small and spend follows the training calendar; scheduled teardown prevents accumulation
- One namespace of identifiers (cohort, space, team, scenario version) spans the token, the telemetry, and the grading assertions, eliminating correlation mapping
- Application teams own their integration contracts as versioned artifacts; the platform team operates the orchestrator, not per-course infrastructure
- Custom build is reduced to three thin artifacts: the integration contract spec and conformance checks, per-app seed and reset hooks, and assertion checkers

### Negative

- Applications must meet a bar to be trainable: team-scoped admin APIs, a seed hook, an audit or event feed, and training-audience token validation; apps without these need remediation before onboarding
- The training/production boundary is cryptographic (issuer and audience validation), not structural; it is only as strong as each app's token validation, which is why contract conformance includes a production-rejection test
- Exercises touching app-global state cannot run in shared apps and take the dedicated-instance exception path, which carries real deployment cost
- Shared apps absorb training load; capacity headroom per concurrent cohort must be planned

## Compliance

Integration contracts are versioned artifacts in the contract registry; an application is onboarded only after its contract passes conformance, which includes demonstrating rejection of a training-audience token by the production deployment. The Reset Controller is the only path that mutates training spaces; direct database writes to training applications are a finding. The dedicated-instance exception is recorded in the Exception Register (GOV-0003) per use. AERB reviews the contract spec and this decision at the standard quarterly review.

## Notes

Architecture diagrams for this decision are maintained as code in `crucible-c4-diagrams.md` (C4 levels 1 through 4, Mermaid) per ADR-0002. The domain model follows ADR-0003; TrainingSpace is the aggregate root. Numbered ADR-0010 on the assumption that ADR-0009 remains reserved for formally establishing the governance system; renumber if that assumption is wrong. Related decisions expected to follow as separate ADRs: the integration contract specification itself (candidate STD), the training-audience token profile, and the assertion definition format.
