# Crucible · C4 Architecture Diagrams

Diagrams-as-code per ADR-0002 (C4 model with Mermaid). Associated with ADR-0010 (Crucible training space model). Four levels: System Context, Container, Component (Orchestrator), and Code (training space lifecycle domain model).

---

## Level 1 · System Context

Crucible orchestrates training inside the already-deployed internal applications. It configures team spaces and simulations through each app's admin API, transforms identity into a training context via the enterprise STS, grades against application state, and publishes results to the LMS.

```mermaid
C4Context
  title Crucible · System Context

  Person(student, "Student", "Employee taking hands-on training in the deployed applications")
  Person(instructor, "Instructor", "Training branch staff running cohorts, injects, resets, and grading")
  Person(appteam, "App Team Engineer", "Owns a training application; publishes its integration contract")

  System(crucible, "Crucible", "Training orchestration and assessment platform: configures training spaces, seeds simulations, resets, and grades")

  System_Ext(apps, "Deployed Training Applications", "Existing internal web apps; expose admin APIs for team spaces and an audit or event feed")
  System_Ext(idp, "Enterprise IdP", "Normal workforce authentication; source of real identity and rosters via groups")
  System_Ext(sts, "Enterprise STS", "Token exchange per ADR-0008; applies the training context transform")
  System_Ext(lms, "LMS", "Training branch system of record for enrollment, grades, and completion")

  Rel(student, crucible, "Takes courses via session cockpit", "HTTPS")
  Rel(instructor, crucible, "Runs cohorts, resets, reviews grading", "HTTPS")
  Rel(appteam, crucible, "Publishes versioned integration contracts", "Git / OCI")
  Rel(student, apps, "Performs exercises in team space", "HTTPS, training-context token")
  Rel(crucible, apps, "Creates, seeds, resets, deletes team spaces; reads audit feed", "Admin API")
  Rel(student, idp, "Authenticates normally", "OIDC")
  Rel(crucible, sts, "Requests training-context token exchange", "OAuth 2.0 Token Exchange")
  Rel(sts, idp, "Validates real identity", "OIDC")
  Rel(crucible, lms, "Publishes grades and completion", "xAPI / LTI")
```

---

## Level 2 · Container

Inside Crucible. Adopted open source carries the portal, scoring, and plumbing; the custom containers are the Orchestrator, the Assertion Engine checkers, and the two registries' contracts.

```mermaid
C4Container
  title Crucible · Container Diagram

  Person(student, "Student")
  Person(instructor, "Instructor")

  System_Boundary(crucible, "Crucible") {
    Container(portal, "Training Portal + Session Cockpit", "Educates", "Cohort access, session lifecycle, instructions, terminals, embedded app view")
    Container(console, "Instructor Console", "Web app", "Cohort setup, live progress, injects, one-click scenario reset, space rebuild, teardown")
    Container(orchestrator, "Orchestrator", "Service", "Configures training spaces through app admin APIs; owns reset, scheduling, and conformance")
    ContainerDb(contracts, "Integration Contract Registry", "OCI registry", "Versioned per-app contracts: space operations, seed hook, event feed pointer")
    ContainerDb(scenarios, "Scenario Pack Registry", "OCI registry", "Versioned simulations: synthetic datasets, app state, accounts, injects")
    Container(assert, "Assertion Engine", "Service + CTFd or PrairieLearn", "State and event assertions scoped by team; weighted rubrics, evidence, overrides")
    Container(bus, "Activity Bus", "NATS JetStream", "Durable stream of student and system events")
    ContainerDb(search, "Activity Search", "OpenSearch", "Indexed audit and telemetry for replay and event assertions")
    Container(lmsbridge, "LMS Bridge", "Service", "Publishes results over xAPI / LTI")
  }

  System_Ext(apps, "Deployed Training Applications", "Admin API + audit feed")
  System_Ext(idp, "Enterprise IdP", "OIDC")
  System_Ext(sts, "Enterprise STS", "Token exchange, ADR-0008")
  System_Ext(lms, "LMS", "System of record")

  Rel(student, portal, "Takes sessions", "HTTPS")
  Rel(instructor, console, "Operates cohorts", "HTTPS")
  Rel(portal, idp, "Authenticates users", "OIDC")
  Rel(portal, sts, "Exchanges for training-context token at session start", "RFC 8693")
  Rel(portal, apps, "Embeds team-scoped app view", "HTTPS")
  Rel(console, orchestrator, "Requests spaces, resets, teardowns", "REST")
  Rel(orchestrator, contracts, "Resolves contract version", "OCI pull")
  Rel(orchestrator, scenarios, "Pulls scenario pack", "OCI pull")
  Rel(orchestrator, apps, "Creates, seeds, resets, deletes team spaces", "Admin API")
  Rel(orchestrator, bus, "Publishes lifecycle events", "NATS")
  Rel(apps, bus, "Streams audit events", "Connector")
  Rel(bus, search, "Indexes activity", "Consumer")
  Rel(assert, apps, "Runs state assertions, scoped by team", "API / DB read")
  Rel(assert, search, "Runs event assertions", "Query")
  Rel(assert, lmsbridge, "Emits scored rubrics", "REST")
  Rel(lmsbridge, lms, "Publishes grades and completion", "xAPI / LTI")
```

---

## Level 3 · Component (Orchestrator)

Inside the Orchestrator, the core custom container. Every mutation of a training space flows through the Space Provisioner against the app's own admin API; nothing writes to app databases directly.

```mermaid
C4Component
  title Crucible Orchestrator · Component Diagram

  Container_Boundary(orch, "Orchestrator") {
    Component(api, "Request API", "REST controller", "Accepts space, reset, and teardown requests from the console and the schedule")
    Component(resolver, "Contract Resolver", "Component", "Fetches and validates the pinned integration contract version for the target app")
    Component(provisioner, "Space Provisioner", "Admin API client", "Executes create, delete, and access operations against the app, per contract")
    Component(seeder, "Seed Runner", "Component", "Applies the scenario pack through the app's seed hook; idempotent re-runs")
    Component(baseline, "Baseline Manifest Store", "Component + DB", "Records object IDs created at setup; the reference for every reset")
    Component(reset, "Reset Controller", "Component", "Executes scenario reset, space rebuild, and cohort teardown as strategies")
    Component(scheduler, "Cohort Scheduler", "Component", "Calendar-driven provisioning and end-date auto-teardown")
    Component(conformance, "Conformance Checker", "Component", "Verifies a contract version against a live app before it is accepted")
    Component(events, "Event Publisher", "Component", "Emits lifecycle events to the activity bus")
  }

  Container_Ext(console, "Instructor Console")
  ContainerDb_Ext(contracts, "Integration Contract Registry")
  ContainerDb_Ext(scenarios, "Scenario Pack Registry")
  System_Ext(apps, "Deployed Training Applications")
  Container_Ext(bus, "Activity Bus")

  Rel(console, api, "Space, reset, teardown requests", "REST")
  Rel(scheduler, api, "Scheduled requests", "Internal")
  Rel(api, resolver, "Resolve contract")
  Rel(resolver, contracts, "Pull pinned version", "OCI")
  Rel(api, provisioner, "Create or delete team spaces")
  Rel(provisioner, apps, "Admin API calls")
  Rel(api, seeder, "Seed simulation")
  Rel(seeder, scenarios, "Pull scenario pack", "OCI")
  Rel(seeder, apps, "Invoke seed hook")
  Rel(seeder, baseline, "Record created object IDs")
  Rel(api, reset, "Execute reset")
  Rel(reset, baseline, "Read baseline manifest")
  Rel(reset, provisioner, "Rebuild via delete + create")
  Rel(reset, seeder, "Re-seed simulation")
  Rel(conformance, apps, "Contract verification calls")
  Rel(api, events, "Lifecycle changes")
  Rel(events, bus, "Publish", "NATS")
```

---

## Level 4 · Code (training space lifecycle domain model)

The domain model behind the Orchestrator, per ADR-0003 (DDD). TrainingSpace is the aggregate root; every reset is a strategy replayed from the contract, the scenario pack version, and the baseline manifest.

```mermaid
classDiagram
  class TrainingSpace {
    +SpaceId id
    +SpaceMode mode
    +SpaceState state
    +CohortId cohort
    +configure()
    +reset(strategy)
    +teardown()
  }
  class SpaceMode {
    <<enumeration>>
    COHORT
    INDIVIDUAL
    EXAM
  }
  class SpaceState {
    <<enumeration>>
    REQUESTED
    CONFIGURED
    SEEDED
    ACTIVE
    RESETTING
    TORN_DOWN
  }
  class IntegrationContract {
    +AppId app
    +SemVer version
    +SpaceOperations operations
    +SeedHook seedHook
    +EventFeedRef eventFeed
    +verifyConformance()
  }
  class SpaceOperations {
    +createSpace(roster)
    +deleteSpace(spaceId)
    +grantAccess(spaceId, principal)
    +revokeAccess(spaceId, principal)
  }
  class SeedHook {
    +apply(spaceId, pack) BaselineManifest
    +idempotent bool
  }
  class ScenarioPack {
    +PackId id
    +SemVer version
    +Dataset[] datasets
    +Inject[] injects
    +RoleTemplate[] roles
  }
  class BaselineManifest {
    +SpaceId space
    +ObjectRef[] createdObjects
    +PackId packVersion
    +Timestamp capturedAt
  }
  class ResetStrategy {
    <<abstract>>
    +execute(space)
  }
  class ScenarioReset {
    +execute(space)
  }
  class SpaceRebuild {
    +execute(space)
  }
  class CohortTeardown {
    +execute(space)
  }
  class Cohort {
    +CohortId id
    +Roster roster
    +Date startDate
    +Date endDate
    +TrainingSpace[] spaces
  }
  class Roster {
    +Principal[] students
    +Principal[] instructors
  }
  class TrainingContextToken {
    +Subject subject
    +ActClaim realIdentity
    +CohortId cohort
    +SpaceId space
    +PackId scenarioVersion
    +Audience audience
    +TTL expiry
  }

  Cohort "1" o-- "many" TrainingSpace
  Cohort "1" *-- "1" Roster
  TrainingSpace "many" --> "1" IntegrationContract : configured via
  TrainingSpace "1" --> "1" ScenarioPack : seeded from
  TrainingSpace "1" *-- "1" BaselineManifest : records
  TrainingSpace "1" --> "1" SpaceMode
  TrainingSpace "1" --> "1" SpaceState
  IntegrationContract "1" *-- "1" SpaceOperations
  IntegrationContract "1" *-- "1" SeedHook
  SeedHook ..> BaselineManifest : produces
  ResetStrategy <|-- ScenarioReset
  ResetStrategy <|-- SpaceRebuild
  ResetStrategy <|-- CohortTeardown
  ScenarioReset ..> SeedHook : re-applies
  SpaceRebuild ..> SpaceOperations : delete + create
  SpaceRebuild ..> SeedHook : re-applies
  CohortTeardown ..> SpaceOperations : delete all
  TrainingContextToken ..> TrainingSpace : scoped to
  TrainingContextToken ..> Cohort : scoped to
```

---

*Diagram source is canonical; rendered exports are disposable. Changes to these diagrams accompany the ADR that motivates them, per ADR-0002.*
