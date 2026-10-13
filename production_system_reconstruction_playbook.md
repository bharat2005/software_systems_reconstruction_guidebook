# Production Software System Reconstruction Playbook

A practical, end-to-end guide to studying an existing production product and independently rebuilding a system with comparable functional behavior and carefully engineered production qualities.

The example throughout is an Instagram-inspired social platform. The objective is **not** to recover Instagram's private source code or guess its exact internal architecture. The objective is to infer externally observable behavior, define an explicit target, and independently build a system that satisfies its own requirements for correctness, security, reliability, maintainability, and performance.

---

## Core principles

1. **Screens are not the system.** A screen is one interface to domain rules, data, workflows, integrations, and operational behavior.
2. **Research from the outside inward; build from the inside outward.** Observe user-visible behavior first. Implement domain rules and data guarantees before completing interfaces and operational layers.
3. **Separate evidence from assumptions.** Label findings as observed, inferred, or unknown.
4. **Specify before implementing.** Important workflows need explicit preconditions, outcomes, authorization rules, and failure behavior.
5. **Design boundaries, then validate them in code.** Avoid both uncontrolled coding and months of speculative upfront design.
6. **Build complete vertical slices.** Prove the architecture through one end-to-end workflow before expanding the feature set.
7. **Correctness comes first.** Authorization, data integrity, transactions, and invariants are foundational.
8. **Failure is normal.** Design for timeouts, retries, duplicate requests, concurrency, partial failures, and recovery.
9. **Use technology to satisfy requirements, not as a substitute for architecture.** Every component should have a clear purpose and operational cost.
10. **Measure quality, not feature count.** A smaller, well-tested and operable system is better than a large collection of fragile screens.

## The complete lifecycle

**Define target → Study reference → Specify requirements → Model domain → Design architecture → Define contracts → Plan milestones → Build vertical slices → Harden → Validate → Deploy → Operate → Compare and iterate.**

The phases below are ordered, but not strictly one-way. New evidence from implementation and testing should feed back into requirements, domain models, and architecture.

---

# Phase 0 — Define the reconstruction target

## Goal

Choose a realistic, bounded target before researching or coding.

## Actions

- Describe the product you intend to build.
- Define the intended users and platforms (web, mobile, or both).
- List in-scope capabilities and explicit non-goals.
- Estimate the expected traffic, data volume, reliability, and security needs.
- Record budget, timeline, technology constraints, and deployment constraints.
- Define what “done” means for the first release.

## Example scope: Instagram-inspired platform

Initial capabilities:
- Accounts, authentication, sessions, and profiles.
- Photo/video posts and media processing.
- Follow/unfollow relationships.
- Likes and comments.
- Home feed and pagination.
- In-app notifications.
- Basic privacy, blocking, and reporting controls.

Possible later capabilities:
- Direct messaging, stories, advanced discovery, ranking/recommendations, large-scale multi-region operation.

Do not attempt to recreate every capability of a mature global platform in the first milestone.

## Deliverable

`docs/project-charter.md`

Include:
- Product objective.
- In-scope capabilities.
- Non-goals.
- Users and roles.
- Constraints and assumptions.
- Initial success criteria.

## Exit criteria

You can explain what you are building, for whom, within what constraints, and what you are deliberately excluding.

---

# Phase 1 — Study the reference product

## Goal

Understand observable behavior rather than copying visual screens.

Use legitimate access to the reference product. Do not bypass access controls or inspect private user data.

## Actions

Create a capability inventory and investigate important workflows.

| Area | Questions to investigate |
|---|---|
| Accounts | Signup, login, logout, session expiry, password recovery |
| Profiles | Editing, visibility, privacy settings |
| Posts | Create, edit, delete, media upload, processing states |
| Feed | Ordering, refresh, pagination, empty states |
| Social graph | Follow requests, unfollow, follower/following lists |
| Engagement | Like/unlike, comments, deletion |
| Notifications | Triggering events, unread state, navigation |
| Search | Query behavior, results, filtering |
| Safety | Blocking, reporting, restricted interactions |
| Settings | Privacy, account controls, preferences |

For each capability, ask:

1. What action does the user perform?
2. What preconditions are required?
3. What state changes?
4. What is shown after success?
5. What happens on failure?
6. What changes for other users?
7. What happens after refresh, logout, or use from another device?
8. Which permissions and privacy rules apply?
9. What edge cases can be explored safely?

Record findings as:
- **Observed:** directly verified behavior.
- **Inferred:** a plausible explanation supported by evidence.
- **Unknown:** not yet verified.

Do not assume a visible behavior reveals the reference product's actual internal implementation.

## Deliverables

- `research/capability-inventory.md`
- `research/user-journeys/`
- `research/observations/`
- `research/open-questions.md`
- Relevant screenshots or notes, where permitted.

## Exit criteria

The important in-scope workflows, expected outcomes, edge cases, and unresolved questions are documented.

---

# Phase 2 — Turn observations into requirements

## Goal

Create a testable specification that will guide implementation and validation.

## Requirement categories

### Functional requirements

What the product must do: publish posts, follow accounts, display feeds, upload media, and notify users.

### Quality requirements

Correctness, security, privacy, availability, latency, durability, accessibility, scalability, and maintainability.

### Constraints

Budget, technology, deployment environment, external dependencies, expected workload, and delivery timeline.

## For each requirement, record

- Unique ID.
- Description and priority.
- Preconditions.
- Expected result and postconditions.
- Authorization rules.
- Failure behavior.
- Acceptance tests.
- Evidence source and confidence.
- Dependencies on other requirements.

Prefer measurable requirements over vague statements.

Weak: “The feed should be fast.”

Better: “Under the documented representative workload, feed requests must meet the selected p95 latency target, use cursor-based pagination, and never return content the viewer is not authorized to access.”

Choose numerical targets based on the intended workload and budget; do not invent targets and treat them as measured guarantees.

## Deliverables

- `requirements/functional-requirements.md`
- `requirements/quality-requirements.md`
- `requirements/acceptance-criteria.md`
- A prioritized backlog and requirements traceability list.

## Exit criteria

Every in-scope capability has explicit, testable behavior and defined priorities.

---

# Phase 3 — Reconstruct the domain model

## Goal

Identify the concepts, relationships, rules, and lifecycles that govern the product.

## Example domain concepts

- User
- Profile
- Follow relationship
- Post
- Media asset
- Like
- Comment
- Notification
- Block relationship
- Feed entry or feed projection

These are candidate concepts, not a mandatory database schema. Some may be combined, split, or implemented as derived projections.

## For every important concept, determine

- Identity and lifecycle.
- Fields and relationships.
- Which domain or module owns it.
- Who can create, read, update, and delete it.
- Business invariants.
- State transitions.
- Concurrency concerns.
- Retention, archival, and deletion behavior.

## Example invariants

- A user cannot follow themselves.
- A user has at most one active follow relationship with a given account.
- A post has an authorized owner.
- A blocked relationship prevents the interactions specified by the product's privacy rules.
- A media asset cannot be published until required upload and processing checks pass.
- Private content is not exposed to unauthorized viewers.

Use database constraints as well as application-level validation where appropriate. Application checks alone do not prevent every race condition.

## Model state machines

For example, a simplified media lifecycle could be:

`PENDING_UPLOAD → UPLOADED → PROCESSING → READY`

with failure and deletion transitions defined explicitly.

Document valid transitions, who can initiate them, and what happens when a transition fails or is retried.

## Identify transaction boundaries

Determine which changes must be atomic. For example, a post record and its initial outbox event may need to be committed in one database transaction. Notification delivery and search indexing typically do not belong in that same database transaction.

## Deliverables

- `docs/domain/glossary.md`
- `docs/domain/entities-and-relationships.md`
- `docs/domain/invariants.md`
- `docs/domain/state-machines.md`
- `docs/domain/context-map.md`

## Exit criteria

You can explain the meaning, ownership, lifecycle, and critical invariants of the core domain concepts.

---

# Phase 4 — Design the system architecture

## Goal

Choose boundaries and components that satisfy requirements without unnecessary complexity.

## Example initial architecture

```text
Web / Mobile Client
        |
        v
      API Layer
        |
        v
   Modular Application
    |       |       |
 Identity  Social  Content
    |       |       |
    +-------+-------+
            |
       PostgreSQL
            |
     Transactional Outbox
            |
     Background Processing
       /       |       \
    Media     Feed   Notifications
       |
  Object Storage
```

This is a possible starting point, not a claim about Instagram's actual architecture.

A modular monolith with a relational database, object storage, and background workers is often a good starting point. Add a dedicated broker, cache, search engine, or separate services when requirements and evidence justify them.

## Decide and document

1. Module boundaries and ownership.
2. Allowed dependencies between modules.
3. Source of truth for each kind of data.
4. Synchronous versus asynchronous operations.
5. Transaction and consistency boundaries.
6. Authentication and authorization boundaries.
7. Failure isolation and degradation behavior.
8. Deployment topology and scaling approach.
9. Observability and operational requirements.
10. Backup, restoration, and disaster recovery approach.

## Architecture decision records (ADRs)

For significant choices, document:
- Context and problem.
- Decision.
- Alternatives considered.
- Benefits and costs.
- Consequences.
- Conditions that would justify revisiting the decision.

Examples: modular monolith versus microservices, relational source of truth, cursor pagination, transactional outbox, or optimistic concurrency control.

## Deliverables

- System context diagram.
- Component/container diagram.
- Deployment diagram.
- Important sequence diagrams.
- Data-flow and async-flow diagrams.
- ADRs.
- `docs/architecture/failure-modes.md`

## Exit criteria

You can trace a major workflow across components and explain ownership, consistency, dependencies, and failure behavior.

---

# Phase 5 — Define API, event, and data contracts

## Goal

Make the interfaces between clients, application modules, storage, and workers explicit before they drift apart.

## API contracts

For each endpoint, define:
- Method and path.
- Request and response schemas.
- Authentication and authorization.
- Validation and stable error formats.
- Pagination, filtering, and sorting.
- Idempotency and retry behavior where needed.
- Concurrency semantics where relevant.
- Rate and resource limits.
- Versioning and compatibility expectations.

Example:

```http
POST /api/v1/posts
Authorization: Bearer <token>
Idempotency-Key: <unique-key>
Content-Type: application/json
```

An idempotency key is useful only if the server defines and enforces its semantics, including how keys, request fingerprints, results, and retention are handled.

## Data contracts

Define:
- Tables, columns, and relationships.
- Foreign keys, uniqueness constraints, and checks.
- Indexes based on actual query patterns.
- Transaction boundaries and isolation requirements.
- Migration and rollback/roll-forward strategy.
- Retention and deletion semantics.
- Data ownership and authoritative versus derived data.

Example follow relationship:

```text
Follow
------
follower_id
followee_id
created_at

Constraints:
- follower_id != followee_id
- unique(follower_id, followee_id)
- foreign keys reference valid accounts
```

## Event contracts

For asynchronous events, define:
- Event type and schema version.
- Event ID and aggregate/resource ID.
- Creation time and relevant metadata.
- Delivery semantics.
- Ordering assumptions.
- Retry and deduplication behavior.
- Consumer compatibility.

Do not promise exactly-once processing unless the full design genuinely provides it. Durable at-least-once delivery with idempotent consumers is a common, practical model.

## Deliverables

- OpenAPI specification.
- Database schema and migration plan.
- Event schemas.
- Contract tests.
- Error and pagination conventions.

## Exit criteria

Frontend, backend, and worker implementation can proceed against stable, explicit contracts.

---

# Phase 6 — Plan implementation order

## Goal

Choose a sequence that respects dependencies and delivers value early.

Example sequence:

1. Engineering foundation.
2. Identity and profiles.
3. Media and posts.
4. Social graph.
5. Likes and comments.
6. Home feed.
7. Notifications.
8. Search and discovery.
9. Advanced features such as messaging, stories, and ranking.

This is a starting order, not a rigid dependency graph. Some work can proceed in parallel once contracts are stable.

## Engineering foundation

Establish:
- Repository and coding conventions.
- Configuration and secrets handling.
- Database migrations.
- Error handling and validation conventions.
- Logging and request correlation.
- Unit and integration test infrastructure.
- CI checks and reproducible local development.
- Basic health checks.

## Deliverables

- Milestone plan.
- Dependency map.
- Prioritized backlog.
- Definition of done for features and milestones.

## Exit criteria

Each milestone has clear dependencies, scope, acceptance criteria, and a reason for its position in the sequence.

---

# Phase 7 — Build the first complete vertical slice

## Goal

Prove that the architecture works end to end before building many features.

Choose one valuable workflow, such as publishing a post.

A possible flow:

```text
User action
    ↓
Frontend
    ↓
API authentication and validation
    ↓
Application use case
    ↓
Domain rules and authorization
    ↓
Database transaction
    ↓
Persist post metadata and outbox event
    ↓
Background processing
    ↓
Update processing/publication state
    ↓
Frontend reflects the result
```

The actual media flow depends on the contract. Large uploads may go directly to object storage through a controlled upload process, with the backend authorizing the upload and verifying the resulting asset.

Implement:
- Domain and application logic.
- Authorization and validation.
- Database constraints and migrations.
- API contract and error handling.
- Transaction and retry safety.
- Background processing where needed.
- Unit, integration, and end-to-end tests.
- Logs, metrics, and request correlation.
- Local setup and deployment instructions.

Test invalid input, unauthorized access, duplicate requests, concurrent operations, worker crashes, and storage failures.

## Deliverable

One complete, tested, observable workflow—not just a screen and a successful HTTP response.

## Exit criteria

The slice satisfies its acceptance criteria and behaves predictably under its defined failure scenarios.

---

# Phase 8 — Repeat the vertical-slice workflow

For every subsequent capability:

1. Review requirements, evidence, and unknowns.
2. Update the domain model and contracts if needed.
3. Implement domain and application logic.
4. Implement persistence and infrastructure.
5. Connect the API and frontend.
6. Test normal behavior and edge cases.
7. Test relevant failure and concurrency scenarios.
8. Add or verify observability.
9. Update documentation and ADRs.
10. Run regression tests and review the change.

Do not hide new requirements inside ad hoc workarounds. If a new discovery invalidates an architectural assumption, revisit the model deliberately.

## Exit criteria

Each feature is integrated into the system without violating established contracts or domain invariants.

---

# Phase 9 — Harden cross-cutting production concerns

These concerns should begin in the first vertical slice and mature as the system grows. This phase is a system-wide hardening review, not a reason to postpone basic safeguards.

| Area | Examples |
|---|---|
| Security | Session security, authorization, input validation, secret management, abuse controls |
| Reliability | Timeouts, bounded retries, idempotency, graceful degradation |
| Data integrity | Transactions, constraints, concurrency control, migrations |
| Async processing | Durable jobs, retry policy, deduplication, dead-letter handling when justified |
| Observability | Structured logs, metrics, traces, dashboards, alerts |
| Performance | Query tuning, pagination, caching where justified, load testing |
| Delivery | CI/CD, environment configuration, deployment checks, rollback |
| Recovery | Backups, restoration drills, retention, disaster recovery |
| Maintainability | Module boundaries, architecture tests, dependency management, documentation |

For each component, document failure modes, detection, mitigation, recovery, and operational ownership.

## Exit criteria

Critical cross-cutting controls are implemented, tested, and consistent across the system.

---

# Phase 10 — Validate production behavior

## Goal

Demonstrate that the implementation meets its requirements instead of relying on assumptions.

## Test layers

- **Unit tests:** domain rules and state transitions.
- **Integration tests:** database, transactions, storage, and workers.
- **Contract tests:** API and event compatibility.
- **End-to-end tests:** important user journeys.
- **Security tests:** unauthorized access, abuse, and common vulnerabilities.
- **Concurrency tests:** simultaneous likes, follows, edits, and retries.
- **Failure tests:** unavailable dependencies, timeouts, and worker crashes.
- **Load tests:** representative traffic, data volume, and concurrency.
- **Migration tests:** upgrading representative existing data.
- **Recovery tests:** restoring data and resuming operations.

## Social-platform checks

- Private content never leaks into unauthorized feeds or search results.
- Blocking and following rules hold under concurrency.
- Duplicate requests do not create unintended duplicate effects.
- Media access respects authorization and deletion rules.
- Failed background jobs can recover without corrupting state.
- Deployments and schema changes preserve supported client compatibility.

## Service-level objectives (SLOs)

Choose measurable targets for:
- API availability.
- Feed latency at p95/p99.
- Background-job completion time.
- Error rates.
- Data recovery objectives.

Targets must match the expected workload and available resources. Measure performance under a documented workload rather than claiming quality based on successful local tests.

## Deliverable

A validation report containing:
- Requirements covered.
- Test results.
- Measured performance.
- Known limitations.
- Unresolved risks.
- Release blockers.

## Exit criteria

Critical correctness, security, reliability, and performance requirements have evidence supporting their acceptance.

---

# Phase 11 — Deploy and operate

## Goal

Make the system reproducibly deployable, diagnosable, and recoverable.

Example delivery pipeline:

```text
Commit
  ↓
CI: lint, tests, security checks, build
  ↓
Versioned artifact
  ↓
Staging deployment
  ↓
Smoke tests and migration validation
  ↓
Production deployment
  ↓
Health checks and SLO monitoring
  ↓
Rollback or mitigation if needed
```

Prepare:
- Reproducible infrastructure and environment configuration.
- Managed secrets and access controls.
- Database backups and verified restoration.
- Health checks, dashboards, and alerts.
- Deployment and rollback procedures.
- Incident response and operational runbooks.
- Safe schema and API evolution procedures.

Deploy to a real environment, observe it, and address issues that were not visible in local development.

## Deliverable

A deployed, monitored system that another engineer can operate using documented procedures.

## Exit criteria

You can deploy, diagnose, recover, and update the system without relying on undocumented manual steps.

---

# Phase 12 — Final parity review and continuous improvement

## Goal

Compare the implementation against the explicit specification and quality targets, not memory or visual similarity alone.

For every in-scope requirement, record one status:
- Implemented and verified.
- Implemented but insufficiently tested.
- Partially implemented.
- Not implemented.
- Blocked by an unresolved decision.

Review:
- Core workflow correctness.
- Privacy and authorization.
- Data ownership and consistency.
- Failure diagnosis and recovery.
- Measured performance.
- Maintainability and ability to evolve.
- Deployment and migration repeatability.
- Documented limitations and trade-offs.

Only declare completion relative to explicit scope and acceptance criteria. A robust implementation can be production-oriented without reproducing every feature or the global operational scale of Instagram.

---

# The daily workflow

## Before implementing a feature

```text
Understand the user journey
        ↓
Review evidence and requirements
        ↓
Identify domain rules and invariants
        ↓
Check data model and ownership
        ↓
Define API and failure contracts
        ↓
Confirm architectural fit
        ↓
Write acceptance tests
```

If an important requirement is unknown, investigate it or record a deliberate, reversible assumption.

## During implementation

```text
Implement a small coherent change
        ↓
Run unit tests
        ↓
Run integration tests
        ↓
Review architectural boundaries
        ↓
Test errors and edge cases
        ↓
Run relevant end-to-end tests
        ↓
Update documentation
```

Use small, reviewable commits. Avoid implementing multiple large, unrelated features simultaneously.

## At the end of each milestone

```text
Validate acceptance criteria
        ↓
Run regression tests
        ↓
Review security and reliability
        ↓
Measure performance where relevant
        ↓
Review debt and unknowns
        ↓
Update architecture documentation
        ↓
Approve the next milestone
```

---

# Suggested repository organization

Keep reference research separate from implementation, either in separate repositories or clearly separated top-level directories.

## Reference research

```text
reference-research/
├── 01-product-inventory/
├── 02-user-journeys/
├── 03-requirements/
├── 04-domain-model/
├── 05-state-machines/
├── 06-api-observations/
├── 07-failure-behavior/
├── 08-architecture-hypotheses/
├── 09-open-questions/
└── 10-acceptance-criteria/
```

## Implementation

```text
social-platform/
├── apps/
│   ├── web/
│   └── api/
├── modules/
│   ├── identity/
│   ├── profiles/
│   ├── social-graph/
│   ├── posts/
│   ├── feed/
│   └── notifications/
├── workers/
├── migrations/
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── contract/
│   ├── e2e/
│   ├── load/
│   └── failure/
├── infrastructure/
├── deploy/
└── docs/
    ├── architecture/
    ├── decisions/
    ├── security/
    └── operations/
```

These are examples, not mandatory layouts. Adapt them to the chosen language, framework, and deployment model.

---

# Starting checklist

Complete these tasks in order before beginning major implementation work.

- [ ] Create the research repository.
- [ ] Write the project charter and define the initial scope.
- [ ] Map the product capabilities.
- [ ] Document five critical journeys: signup/login, publish a post, follow a user, like/comment, and view the feed.
- [ ] Investigate edge cases, permissions, privacy, failures, and concurrent actions.
- [ ] Write testable functional and quality requirements.
- [ ] Create the initial domain model, relationships, invariants, and state machines.
- [ ] Draw the initial architecture and data flows.
- [ ] Define the first vertical slice, contracts, tests, and acceptance criteria.
- [ ] Implement and validate that slice before expanding the feature set.

Do not wait until every future requirement is known. Design enough to make the first slice sound, then refine the architecture based on evidence and implementation.

---

# Definition of done

A feature is not complete merely because its screen works. For each feature, evaluate the applicable items below:

- [ ] Requirements and acceptance criteria are satisfied.
- [ ] Domain invariants are enforced.
- [ ] Authentication and authorization are correct.
- [ ] Invalid input and errors are handled.
- [ ] Transactions and concurrency behavior are understood.
- [ ] Retries and duplicate requests are safe where required.
- [ ] Failure behavior is defined and tested.
- [ ] Data lifecycle and deletion behavior are handled.
- [ ] Unit, integration, and relevant end-to-end tests exist.
- [ ] Logs, metrics, and traces are sufficient to diagnose failures.
- [ ] Security and privacy implications are addressed.
- [ ] Migrations and deployment behavior are understood.
- [ ] Performance meets applicable targets.
- [ ] Documentation and operational procedures are updated.

Not every feature needs every control at equal depth, but omissions should be deliberate and risk-based.

# Final operating principles

1. Never equate screens with features or features with systems.
2. Never begin a major feature without understanding its requirements.
3. Design boundaries before implementation, then validate them through code.
4. Prefer a complete vertical slice over many incomplete features.
5. Treat authorization, transactions, and data integrity as foundational.
6. Design for retries, timeouts, concurrency, partial failures, and recovery.
7. Add infrastructure only when a requirement justifies its benefits and cost.
8. Keep observed facts separate from architectural hypotheses.
9. Make testing, observability, security, and operations part of development.
10. Optimize for verified quality, not feature count.

**The repeatable development loop is:**

**Study → Specify → Model → Architect → Contract → Plan → Implement → Test → Harden → Deploy → Operate → Validate → Iterate.**
