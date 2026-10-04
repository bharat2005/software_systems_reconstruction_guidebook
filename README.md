# Production App Reverse-Engineering & Rebuild Playbook

> A repeatable workflow for independently reverse-engineering a real production application and rebuilding a functionally similar, robust, maintainable, production-grade system from scratch.

---

# 0. The Goal

The goal is **not**:

> Recreate the same screens as quickly as possible.

The goal is:

> Reconstruct the important **behavior, workflows, domain rules, state management, data ownership, failure handling, lifecycle behavior, architecture and quality characteristics** of the reference application.

Think:

```text
Reference App
     ↓
Observe
     ↓
Understand
     ↓
Model
     ↓
Design
     ↓
Implement
     ↓
Validate
     ↓
Harden
     ↓
Production-Ready Rebuild
```

---

# 1. Core Principles

These rules govern the entire project.

## 1.1 Screens are not the architecture

Do not think:

```text
Home
Product
Cart
Checkout
```

Think:

```text
Business Capabilities
        ↓
Workflows
        ↓
Domain Rules
        ↓
State
        ↓
Data
        ↓
UI
```

---

## 1.2 Do not code immediately

The first job is to understand the reference system.

Avoid:

```text
Open Android Studio
↓
Build Home Screen
↓
Build Product Screen
↓
Build Cart Screen
↓
Architecture starts collapsing
```

Instead:

```text
Explore
↓
Document
↓
Model
↓
Architect
↓
Implement
```

---

## 1.3 Build vertical slices

Never build the entire UI first, then backend/data, then integration.

Instead:

```text
Feature
 ├── Domain
 ├── Data
 ├── Presentation
 ├── Persistence
 ├── Error handling
 └── Tests
```

Complete one meaningful capability end-to-end before moving on.

---

## 1.4 Every important state needs an owner

For every important piece of state, know:

```text
Who owns it?
Who can modify it?
Who can observe it?
What is the source of truth?
Where is it persisted?
How can it become stale?
How is it synchronized?
```

Avoid multiple competing copies of the same state.

---

## 1.5 Production quality means handling failure

Do not only implement:

```text
Happy path
```

Also consider:

```text
No network
Timeout
Server error
Unauthorized
Expired session
Duplicate action
Out-of-order responses
App restart
Process death
Background/foreground
Stale data
Database failure
Malformed response
```

---

## 1.6 Preserve essential complexity, eliminate accidental complexity

Keep complexity that comes from the problem:

```text
Business rules
Authentication
Caching
Synchronization
Concurrency
Persistence
Offline behavior
Payments
Security
```

Avoid complexity created by your implementation:

```text
Unnecessary abstractions
Useless interfaces
Giant Common modules
Global mutable state
Duplicate state
Deep forwarding layers
```

---

# 2. Complete Project Lifecycle

The complete workflow is:

```text
PHASE 1   Choose Reference
    ↓
PHASE 2   Explore Reference
    ↓
PHASE 3   Extract Requirements
    ↓
PHASE 4   Model Domain & System
    ↓
PHASE 5   Design Architecture
    ↓
PHASE 6   Build Foundation
    ↓
PHASE 7   Implement Vertical Slices
    ↓
PHASE 8   Integrate Entire System
    ↓
PHASE 9   Production Hardening
    ↓
PHASE 10  Final Validation & Release
```

Do not skip phases merely because you are eager to start coding.

---

# 3. Phase 1 — Choose the Reference Application

## Goal

Select a real production application and define exactly what you are going to reproduce.

---

## Tasks

* [ ] Select reference application.
* [ ] Install/use the production application.
* [ ] Identify its major capabilities.
* [ ] Estimate complexity.
* [ ] Define your rebuild scope.
* [ ] Decide platform.
* [ ] Decide whether you will build a backend.
* [ ] Decide whether external services will be real, mocked or replaced.
* [ ] Define what "production-grade" means for this project.
* [ ] Define what is explicitly out of scope.

---

## Scope document

Create:

```text
docs/planning/scope.md
```

Record:

```text
Reference App:
Target Platform:
Target Architecture:
Backend:
Database:
Authentication:
External Services:
In-Scope Features:
Out-of-Scope Features:
Performance Goals:
Quality Goals:
```

---

## Important

Do not attempt to reproduce infrastructure that cannot reasonably be observed.

You are aiming for:

```text
Behavioral equivalence
+
Robust engineering properties
```

not:

```text
Identical source code
```

---

# 4. Phase 2 — Explore & Reverse Engineer the Reference

## Goal

Understand what the application actually does.

Do not make architectural decisions yet.

You are collecting evidence.

---

# 4.1 Build a Screen Inventory

Create:

```text
docs/discovery/screen-inventory.md
```

Record:

```text
Screen
Purpose
Entry points
Exit points
Data displayed
Actions
Loading state
Empty state
Error state
Permissions
Persistence
Related features
```

Example:

```text
Product Details

Entry:
Search
Category
Deep link

Displays:
Product
Price
Availability
Images
Reviews

Actions:
Add to cart
Favorite
Share

States:
Loading
Loaded
Unavailable
Error
```

---

# 4.2 Build a Feature Inventory

Do NOT organize only by screens.

Identify business capabilities.

Example:

```text
Authentication
 ├── Login
 ├── Logout
 ├── Session restoration
 └── Token refresh

Catalog
 ├── Browse
 ├── Search
 ├── Filtering
 ├── Sorting
 └── Product details

Cart
 ├── Add item
 ├── Remove item
 ├── Quantity
 └── Persistence

Checkout
 ├── Address
 ├── Payment
 ├── Order creation
 └── Confirmation
```

Create:

```text
docs/discovery/feature-inventory.md
```

---

# 4.3 Explore User Journeys

For every major feature, determine:

```text
Entry
 ↓
Action
 ↓
System behavior
 ↓
State change
 ↓
Next action
 ↓
Completion
```

Example:

```text
Browse
 ↓
Product
 ↓
Add to Cart
 ↓
Cart Updated
 ↓
Checkout
 ↓
Payment
 ↓
Order Created
 ↓
Order Confirmation
```

Create:

```text
docs/discovery/user-journeys.md
```

---

# 4.4 Explore Alternate Paths

Do not only test the happy path.

For every important workflow test:

```text
Success
Failure
Cancellation
Retry
Back navigation
Refresh
App restart
Network unavailable
Slow network
Duplicate action
Background/foreground
```

---

# 4.5 Perform Black-Box Experiments

Treat the reference application like a laboratory.

Ask questions such as:

```text
Does cart survive restart?
Does search work offline?
Is data cached?
What happens when authentication expires?
What happens when a request times out?
What happens if the user taps twice?
Does the operation retry?
What happens after process death?
```

Design experiments to answer these questions.

---

# 4.6 Record Unknowns

Never silently guess.

Create:

```text
docs/discovery/unknowns.md
```

Classify observations as:

```text
CONFIRMED
LIKELY
ASSUMPTION
UNKNOWN
```

Example:

```text
[CONFIRMED]
Cart survives application restart.

[LIKELY]
Product data is locally cached.

[UNKNOWN]
Whether cart mutations are queued offline.

[ASSUMPTION]
Server is authoritative for product price.
```

---

# 4.7 Discovery Exit Criteria

Do not leave Phase 2 until you can explain:

* [ ] Major screens
* [ ] Major capabilities
* [ ] Critical user journeys
* [ ] Important alternate flows
* [ ] Loading/empty/error states
* [ ] Major persistence behavior
* [ ] Important lifecycle behavior
* [ ] Major unknowns
* [ ] High-risk areas

---

# 5. Phase 3 — Extract Requirements

## Goal

Turn observations into an implementable specification.

At this point you should stop thinking primarily in terms of screens.

---

# 5.1 Functional Requirements

Create:

```text
docs/requirements/functional-requirements.md
```

Example:

```text
AUTH-001
User can authenticate with valid credentials.

AUTH-002
Invalid credentials produce an actionable error.

AUTH-003
Authenticated session survives application restart.

AUTH-004
Expired credentials are refreshed when possible.

AUTH-005
User can explicitly log out.
```

---

# 5.2 Business Rules

Create:

```text
docs/requirements/business-rules.md
```

Example:

```text
Cart quantity must be > 0.

An unavailable product cannot be purchased.

An order cannot transition from COMPLETED back to PENDING.

A user cannot access another user's order.

Payment amount must equal the final checkout amount.
```

---

# 5.3 Acceptance Criteria

For every capability define:

```text
Given
When
Then
```

Example:

```text
Given the user is authenticated
When the user adds an available product to the cart
Then the cart contains the product
And the updated quantity is visible
And the state survives navigation.
```

---

# 5.4 Requirements Exit Criteria

For every major feature:

* [ ] Functional behavior defined
* [ ] Business rules defined
* [ ] Success behavior defined
* [ ] Failure behavior defined
* [ ] Important edge cases defined
* [ ] Acceptance criteria defined
* [ ] Unknown assumptions documented

---

# 6. Phase 4 — Domain & System Modeling

## Goal

Understand the underlying system before deciding how to implement it.

---

# 6.1 Identify Domain Entities

Examples:

```text
User
Session
Product
Category
Cart
CartItem
Address
Payment
Order
OrderItem
Coupon
Notification
```

Do not create entities merely because you saw a screen.

Create them because they represent meaningful concepts or state.

---

# 6.2 Identify Relationships

Example:

```text
User
 ├── Sessions
 ├── Addresses
 ├── Cart
 └── Orders

Cart
 └── CartItems
       └── Product

Order
 ├── OrderItems
 ├── Payment
 └── Address
```

---

# 6.3 Identify Ownership

For every important entity:

```text
Who owns it?
Who can mutate it?
Who can read it?
Who validates it?
```

Example:

```text
Cart
Owner: Cart domain

Order
Owner: Order domain

Product
Owner: Catalog domain
```

---

# 6.4 Identify Source of Truth

For every important piece of data:

```text
Server?
Local database?
Memory?
Combination?
```

Example:

```text
Product price
    ↓
Server authoritative
    ↓
Local cache is a representation
```

---

# 6.5 Identify Invariants

Ask:

> What must NEVER become invalid?

Examples:

```text
Quantity > 0

Order state transitions must be valid.

User can only access authorized resources.

Money calculations must be exact.

A completed payment cannot accidentally become unpaid.
```

Create:

```text
docs/design/invariants.md
```

---

# 6.6 Model Important State Machines

Examples:

```text
Authentication:

LoggedOut
    ↓
Authenticating
    ↓
Authenticated
    ↓
Refreshing
    ↓
Authenticated / LoggedOut
```

Payment:

```text
Idle
 ↓
Creating
 ├── Success
 ├── Failed
 └── Unknown
       ↓
    Verifying
```

Create:

```text
docs/design/state-machines/
```

---

# 6.7 Model Data Flow

For important workflows:

```text
UI
 ↓
Presentation
 ↓
Use Case
 ↓
Repository
 ↓
Local / Remote Data Source
 ↓
Persistence / Server
```

Then determine the actual flow required by the feature.

---

# 6.8 Domain Modeling Exit Criteria

You should be able to answer:

* [ ] What are the core entities?
* [ ] Who owns them?
* [ ] What rules govern them?
* [ ] What are their important states?
* [ ] What transitions are valid?
* [ ] Where is the source of truth?
* [ ] What data must persist?
* [ ] What happens when local and remote state disagree?

---

# 7. Phase 5 — Architecture Design

## Goal

Translate your domain understanding into maintainable technical boundaries.

---

# 7.1 Start With Boundaries

Think:

```text
Application
    ↓
Features / Capabilities
    ↓
Domain
    ↓
Data
    ↓
Infrastructure
```

A possible Android structure:

```text
app/

core/
    common/
    network/
    database/
    security/
    logging/
    testing/

domain/
    auth/
    catalog/
    cart/
    checkout/
    orders/

data/
    auth/
    catalog/
    cart/
    checkout/
    orders/

feature/
    auth/
    home/
    search/
    product/
    cart/
    checkout/
    orders/
```

This is only a starting point.

Do not blindly reproduce this structure.

---

# 7.2 Define Dependency Direction

Example:

```text
UI
 ↓
Presentation
 ↓
Domain
 ↓
Data abstractions
 ↓
Data implementations
 ↓
Infrastructure
```

Avoid accidental dependencies such as:

```text
Domain → Android UI
Domain → Navigation
Domain → Retrofit
Domain → Room
```

unless there is a deliberate reason.

---

# 7.3 Define State Ownership

For every shared state:

```text
Owner:
Source of truth:
Readers:
Writers:
Persistence:
Synchronization:
Invalidation:
```

---

# 7.4 Define Data Strategy

For each data source decide:

```text
Remote source
Local source
Cache
Synchronization
Retry
Pagination
Offline behavior
Conflict handling
```

Do not automatically make everything offline-first.

Make the decision based on requirements.

---

# 7.5 Define Error Strategy

Classify important errors:

```text
Network unavailable
Timeout
Unauthorized
Forbidden
Not found
Validation failure
Conflict
Rate limited
Server failure
Malformed response
Local storage failure
Unknown
```

Define appropriate recovery behavior.

---

# 7.6 Define Security Strategy

Consider:

```text
Authentication
Authorization
Credential storage
Sensitive data
Logging redaction
Transport security
Session expiration
Logout
```

---

# 7.7 Define Testing Strategy

Decide what belongs in:

```text
Unit tests
Integration tests
Repository/data tests
UI tests
End-to-end tests
```

Focus testing around behavior and failure modes, not merely code coverage.

---

# 7.8 Record Important Architectural Decisions

Create:

```text
docs/architecture/adr/
```

Example:

```text
ADR-001-cart-source-of-truth.md
ADR-002-authentication-strategy.md
ADR-003-offline-strategy.md
```

For each:

```text
Decision
Context
Alternatives
Trade-offs
Consequences
```

---

# 7.9 Architecture Exit Criteria

Before implementation, you should understand:

* [ ] Major modules/boundaries
* [ ] Dependency direction
* [ ] Domain ownership
* [ ] State ownership
* [ ] Data sources
* [ ] Persistence
* [ ] Caching
* [ ] Error strategy
* [ ] Authentication/security
* [ ] Testing strategy
* [ ] Major architectural decisions

You do NOT need to know every class you will ever create.

---

# 8. Phase 6 — Build the Project Foundation

## Goal

Create the infrastructure required for feature development.

---

# 8.1 Project Setup

* [ ] Android project
* [ ] Kotlin configuration
* [ ] Build variants
* [ ] Dependency management
* [ ] Module structure
* [ ] Formatting
* [ ] Static analysis
* [ ] Lint

---

# 8.2 Core Infrastructure

Implement only what the application actually needs:

* [ ] Networking
* [ ] Serialization
* [ ] Database
* [ ] Secure storage
* [ ] Dependency injection
* [ ] Navigation foundation
* [ ] Logging
* [ ] Error model
* [ ] Configuration
* [ ] Testing infrastructure

---

# 8.3 CI

At minimum:

```text
Build
 ↓
Unit Tests
 ↓
Lint / Static Analysis
 ↓
Package
```

Automate repetitive validation as early as practical.

---

# 8.4 Foundation Exit Criteria

The application should:

* [ ] Compile
* [ ] Launch
* [ ] Navigate through basic shell
* [ ] Execute a sample network operation if applicable
* [ ] Access persistence if applicable
* [ ] Run tests
* [ ] Pass static checks
* [ ] Produce a reproducible build

---

# 9. Phase 7 — Implement Features Using Vertical Slices

This is where most of your development time will go.

---

# 9.1 Choose Feature Order Strategically

Do not simply build screens in visual order.

Prioritize:

```text
Dependencies
+
Architectural risk
+
Business importance
```

Example:

```text
Authentication
    ↓
Catalog
    ↓
Search
    ↓
Product
    ↓
Cart
    ↓
Checkout
    ↓
Orders
```

Actual order depends on your system.

---

# 9.2 Feature Implementation Workflow

For every feature:

```text
1. Understand
2. Specify
3. Model
4. Implement
5. Test
6. Integrate
7. Harden
8. Mark Done
```

---

## Step 1 — Understand

Before coding:

* [ ] Reference behavior understood
* [ ] Requirements understood
* [ ] Dependencies identified
* [ ] Unknowns resolved

---

## Step 2 — Specify

Define:

```text
Inputs
Outputs
States
Business rules
Failures
Acceptance criteria
```

---

## Step 3 — Implement Domain

Implement:

```text
Entities
Value objects
Business rules
Use cases
State transitions
Contracts
```

Do not put UI concerns here.

---

## Step 4 — Implement Data

Implement what the feature requires:

```text
API
Repository
Database
Cache
Mapping
Synchronization
Retry
Persistence
```

---

## Step 5 — Implement Presentation

Implement:

```text
UI state
ViewModel/presentation logic
Screen
Navigation
User interactions
Loading state
Empty state
Error state
Success state
```

UI should primarily represent application state rather than become the owner of business truth.

---

## Step 6 — Integrate

Connect the feature to existing capabilities.

Check:

```text
State sharing
Navigation
Data consistency
Authentication
Persistence
Cross-feature behavior
```

---

## Step 7 — Test

Test:

```text
Happy path
Failure path
Edge cases
Concurrency
Lifecycle
Persistence
Integration
```

---

## Step 8 — Harden

Ask:

```text
What if the network disappears?
What if the user taps twice?
What if the request times out?
What if the app is killed?
What if the response arrives late?
What if data is stale?
What if authentication expires?
```

Fix the important problems.

---

# 10. Feature Definition of Done

Never mark a feature complete merely because the screen works.

A feature is DONE when:

### Requirements

* [ ] Functional requirements satisfied
* [ ] Acceptance criteria satisfied

### Domain

* [ ] Business rules implemented
* [ ] Invariants protected
* [ ] State transitions correct

### Data

* [ ] Source of truth defined
* [ ] Persistence correct
* [ ] API handling correct
* [ ] Cache behavior correct where required
* [ ] Retry behavior correct where required

### Presentation

* [ ] Loading state
* [ ] Empty state
* [ ] Error state
* [ ] Success state
* [ ] Lifecycle behavior

### Reliability

* [ ] Failure cases considered
* [ ] Concurrency considered
* [ ] Duplicate actions handled
* [ ] Process death considered

### Testing

* [ ] Unit tests
* [ ] Integration tests where appropriate
* [ ] UI/E2E tests for critical journeys

### Architecture

* [ ] Correct module
* [ ] Correct dependency direction
* [ ] No unnecessary coupling
* [ ] No unexplained hacks

---

# 11. Phase 8 — Integrate the Entire System

## Goal

Ensure that independently working features actually form one coherent system.

---

# 11.1 Test Complete User Journeys

Example:

```text
Launch
 ↓
Login
 ↓
Browse
 ↓
Search
 ↓
Product
 ↓
Add to Cart
 ↓
Cart
 ↓
Checkout
 ↓
Payment
 ↓
Order
 ↓
Order Details
 ↓
Logout
```

---

# 11.2 Test Cross-Feature State

Verify:

```text
Cart changes appear everywhere expected.

Authentication state is consistent.

Product changes don't corrupt cart.

Order creation updates the expected state.

Logout clears the appropriate state.

Cached data doesn't leak across accounts.
```

---

# 11.3 Integration Tests

Test critical combinations:

```text
Authentication + API
Catalog + Database
Catalog + Cache
Cart + Catalog
Checkout + Payment
Order + Authentication
```

---

# 11.4 Integration Exit Criteria

* [ ] Critical journeys complete successfully
* [ ] Shared state is consistent
* [ ] Navigation is correct
* [ ] Authentication boundaries work
* [ ] Persistence works
* [ ] Major integration bugs resolved

---

# 12. Phase 9 — Production Hardening

Now deliberately try to break your application.

---

# 12.1 Network

Test:

```text
[ ] Offline
[ ] Slow network
[ ] Timeout
[ ] Connection reset
[ ] 401
[ ] 403
[ ] 404
[ ] 409
[ ] 429
[ ] 500
[ ] Malformed response
```

---

# 12.2 Concurrency

Test:

```text
[ ] Double tap
[ ] Multiple requests
[ ] Out-of-order responses
[ ] Concurrent refresh
[ ] Concurrent mutations
[ ] Retry after timeout
```

Ask:

> Is this operation idempotent?

---

# 12.3 Lifecycle

Test:

```text
[ ] Rotation/configuration change
[ ] Background
[ ] Foreground
[ ] Process death
[ ] App restart
[ ] Navigation away during request
```

---

# 12.4 Persistence

Test:

```text
[ ] Cold start
[ ] Database restart
[ ] Migration
[ ] Stale cache
[ ] Corrupt/unexpected data
[ ] Logout/login with different user
```

---

# 12.5 Security

Verify:

```text
[ ] Credentials aren't logged
[ ] Sensitive data isn't stored insecurely
[ ] Authorization is enforced
[ ] Expired sessions are handled
[ ] Logout behaves correctly
[ ] Sensitive analytics/events are not leaked
```

---

# 12.6 Performance

Measure important areas:

```text
[ ] Startup
[ ] Screen rendering
[ ] Scrolling
[ ] Large lists
[ ] Database queries
[ ] Network usage
[ ] Memory
[ ] Image loading
[ ] Battery/background work
```

Optimize based on evidence rather than guessing.

---

# 12.7 Accessibility

Verify:

```text
[ ] Content descriptions
[ ] Screen reader navigation
[ ] Touch target sizes
[ ] Font scaling
[ ] Contrast
[ ] Keyboard/navigation support where relevant
```

---

# 12.8 Observability

Have enough visibility to answer:

```text
What failed?
Where?
For which operation?
Under what conditions?
Can I reproduce it?
```

Use appropriate:

```text
Logging
Metrics
Crash reporting
Analytics
Tracing
```

Do not log sensitive information.

---

# 13. Phase 10 — Reference-vs-Rebuild Validation

This is especially important for your challenge.

Go back to the reference application.

For every major capability compare:

```text
Reference
      VS
Your App
```

Compare:

### Behavior

```text
[ ] Same expected workflows
[ ] Same important state transitions
[ ] Same error behavior
[ ] Same persistence behavior
[ ] Same navigation behavior
```

### UX

```text
[ ] Similar interaction model
[ ] Similar loading states
[ ] Similar empty states
[ ] Similar error states
```

### System properties

```text
[ ] Similar caching characteristics
[ ] Similar offline behavior where observable
[ ] Similar retry behavior
[ ] Similar lifecycle behavior
```

Don't obsess over internal implementation equivalence.

You are validating **observable behavior and engineering quality**.

---

# 14. Phase 11 — Final Production Readiness

Before declaring the project finished:

## Functionality

* [ ] All in-scope capabilities implemented
* [ ] Critical journeys working
* [ ] Acceptance criteria satisfied

## Architecture

* [ ] Boundaries make sense
* [ ] Dependencies are intentional
* [ ] State ownership is clear
* [ ] No major architectural hacks

## Reliability

* [ ] Failure scenarios tested
* [ ] Concurrency considered
* [ ] Lifecycle behavior tested
* [ ] Persistence validated

## Security

* [ ] Authentication secure
* [ ] Authorization correct
* [ ] Sensitive data protected
* [ ] Sensitive logging removed

## Performance

* [ ] Startup acceptable
* [ ] Major screens performant
* [ ] Memory acceptable
* [ ] Database/network behavior reasonable

## Testing

* [ ] Unit tests
* [ ] Integration tests
* [ ] Critical UI/E2E tests
* [ ] Regression tests

## Release

* [ ] Release build works
* [ ] Real-device testing completed
* [ ] Configuration verified
* [ ] Signing configured
* [ ] Known limitations documented

---

# 15. The Daily Development Loop

Once you reach implementation, your daily loop becomes very simple:

```text
                 ┌──────────────┐
                 │ Choose Task  │
                 └──────┬───────┘
                        ↓
                 ┌──────────────┐
                 │ Understand   │
                 │ Requirement │
                 └──────┬───────┘
                        ↓
                 ┌──────────────┐
                 │ Implement    │
                 └──────┬───────┘
                        ↓
                 ┌──────────────┐
                 │ Test         │
                 └──────┬───────┘
                        ↓
                 ┌──────────────┐
                 │ Integrate    │
                 └──────┬───────┘
                        ↓
                 ┌──────────────┐
                 │ Harden       │
                 └──────┬───────┘
                        ↓
                 ┌──────────────┐
                 │ Mark Done    │
                 └──────┬───────┘
                        ↓
                   Next Task
```

---

# 16. The Architecture Review Loop

Every few features, stop implementation temporarily.

Ask:

### State

```text
Where is the source of truth?
```

### Ownership

```text
Who owns this behavior?
```

### Dependencies

```text
Why does A depend on B?
```

### Failure

```text
What happens if this operation fails halfway?
```

### Concurrency

```text
What happens if this happens twice simultaneously?
```

### Lifecycle

```text
What happens if Android kills the process here?
```

### Persistence

```text
What survives restart?
```

### Consistency

```text
What happens if local and remote state disagree?
```

### Evolution

```text
How difficult would it be to add the next feature?
```

If these questions become increasingly difficult to answer, stop adding features and fix the architecture.

---

# 17. The "Change Test"

Periodically invent hypothetical requirements.

Example:

```text
What if Product now supports subscriptions?
```

Then:

```text
How many modules change?
How many screens change?
How many repositories change?
Does unrelated functionality break?
```

Another:

```text
What if Checkout supports another payment provider?
```

Another:

```text
What if Product becomes available offline?
```

Another:

```text
What if the API changes from v1 to v2?
```

Healthy architecture makes localized changes possible.

If one requirement requires touching unrelated areas throughout the codebase, investigate your boundaries.

---

# 18. The "Delete the UI" Test

Ask:

> If I deleted all screens, would the business system still make conceptual sense?

You should still have meaningful concepts such as:

```text
Cart
Order
Payment
Authentication
Catalog
```

The UI should be a projection of the application rather than the place where the business system lives.

---

# 19. The "Source of Truth" Test

For every important state ask:

```text
Where does truth live?
```

Example:

```text
Product price
→ Server

Cached product
→ Local representation

UI
→ Projection
```

Avoid:

```text
Server price
+
Repository price
+
ViewModel price
+
Screen price
+
Global singleton price
```

unless there is a deliberate reason and clear synchronization semantics.

---

# 20. The "Failure First" Test

For every important operation ask:

```text
What happens when it succeeds?
What happens when it fails?
What happens when it times out?
What happens when it is repeated?
What happens when the app dies?
What happens when the network disappears?
```

This should become second nature.

---

# 21. Project Documentation Structure

Recommended starting structure:

```text
project/
│
├── docs/
│   │
│   ├── planning/
│   │   ├── scope.md
│   │   ├── constraints.md
│   │   └── success-criteria.md
│   │
│   ├── discovery/
│   │   ├── screen-inventory.md
│   │   ├── feature-inventory.md
│   │   ├── user-journeys.md
│   │   ├── observations.md
│   │   ├── error-scenarios.md
│   │   └── unknowns.md
│   │
│   ├── requirements/
│   │   ├── functional-requirements.md
│   │   ├── business-rules.md
│   │   └── acceptance-criteria.md
│   │
│   ├── design/
│   │   ├── domain-model.md
│   │   ├── entity-relationships.md
│   │   ├── data-ownership.md
│   │   ├── invariants.md
│   │   └── state-machines/
│   │
│   ├── architecture/
│   │   ├── system-architecture.md
│   │   ├── module-structure.md
│   │   ├── dependency-graph.md
│   │   ├── data-flow.md
│   │   ├── error-strategy.md
│   │   └── adr/
│   │
│   ├── testing/
│   │   ├── test-strategy.md
│   │   ├── test-scenarios.md
│   │   └── production-failures.md
│   │
│   └── release/
│       └── production-readiness.md
│
├── app/
├── core/
├── domain/
├── data/
├── feature/
│
└── README.md
```

Don't create all of these files on day one. Create them as their corresponding phase begins.

---

# 22. Recommended Task Board

Track work by **capability**, not merely screens.

Example:

```text
BACKLOG
────────────────────────────
AUTH-001 Session model
AUTH-002 Login
AUTH-003 Session persistence
AUTH-004 Token refresh

CAT-001 Product model
CAT-002 Product API
CAT-003 Product persistence
CAT-004 Product details

CART-001 Cart model
CART-002 Add item
CART-003 Remove item
CART-004 Cart persistence
```

Then:

```text
READY
IN PROGRESS
BLOCKED
DONE
```

---

# 23. Risk-Driven Development

Don't ask:

> "What's the easiest feature to build?"

Ask:

> "What uncertainty or architectural risk could hurt me later?"

Prioritize high-risk capabilities early.

Typical high-risk areas:

```text
Authentication
Payments
Synchronization
Offline mutations
Complex caching
Large datasets
Background work
Concurrency
File uploads
Real-time state
```

Discover architectural problems while the codebase is still small.

---

# 24. Reusable Platform Across Multiple Rebuilds

After several projects, identify infrastructure that genuinely repeats:

```text
Networking
Database
Logging
Security
Analytics
Testing utilities
Configuration
Build conventions
Design system
```

Extract proven components into your own reusable platform.

Do NOT build a giant framework before your first application.

Use:

```text
App 1
 ↓
Discover repetition
 ↓
Extract

App 2
 ↓
Discover improvement
 ↓
Extract

App 3
 ↓
Mature platform
```

This is how you gain massive leverage as a solo developer.

---

# 25. What NOT To Do

## Don't do this

```text
Screen
 ↓
ViewModel
 ↓
API

Next Screen
 ↓
Another ViewModel
 ↓
Another API
```

---

## Don't build all UI first

```text
All screens
 ↓
All ViewModels
 ↓
All repositories
 ↓
Integration nightmare
```

---

## Don't create architecture just to look professional

Avoid:

```text
Screen
 ↓
ViewModel
 ↓
UseCase
 ↓
Interactor
 ↓
Manager
 ↓
Repository
 ↓
DataSource
```

if every layer simply forwards the same call.

---

## Don't create a giant Common module

Avoid:

```text
common/
 ├── utils
 ├── managers
 ├── helpers
 ├── models
 └── everything-else
```

Shared code should have a reason to be shared.

---

## Don't use global mutable state as glue

If you find yourself thinking:

> "I'll just put this in a singleton."

Stop and determine who actually owns the state.

---

## Don't silently accept hacks

If you temporarily need one:

```text
TODO
TECH-DEBT
```

Record it.

Know whether the workaround is temporary or an intentional architectural decision.

---

# 26. The Most Important Questions

For every significant feature, you should eventually be able to answer:

### Behavior

> What does the feature do?

### Domain

> What business concept does it represent?

### Ownership

> Who owns its state?

### Authority

> Where is the source of truth?

### Data

> Where does its data come from?

### Persistence

> What survives restart?

### Failure

> What happens when it fails?

### Concurrency

> What happens when it happens twice?

### Lifecycle

> What happens if the app dies?

### Security

> Who is allowed to perform it?

### Testing

> How do I know it works?

### Evolution

> How would I add the next requirement?

If you cannot answer these, investigate before adding more complexity.

---

# 27. The Mental Model to Remember

When you see:

```text
Home Screen
```

don't think:

```text
Build HomeScreen.kt
```

Think:

```text
What capability does Home expose?

What state does it display?

Where does that state come from?

Who owns it?

What workflows modify it?

What business rules apply?

What happens if the data is stale?

What happens offline?

What happens during process death?

What other capabilities depend on it?
```

The screen is only the final projection.

---

# 28. The Entire Method in One Page

```text
┌─────────────────────────────────────────────┐
│ PHASE 1 — CHOOSE                           │
│ Reference app + scope + success criteria   │
└──────────────────────┬──────────────────────┘
                       ↓
┌─────────────────────────────────────────────┐
│ PHASE 2 — EXPLORE                          │
│ Screens + features + journeys + failures   │
│ + lifecycle + unknowns                     │
└──────────────────────┬──────────────────────┘
                       ↓
┌─────────────────────────────────────────────┐
│ PHASE 3 — SPECIFY                          │
│ Requirements + rules + acceptance criteria │
└──────────────────────┬──────────────────────┘
                       ↓
┌─────────────────────────────────────────────┐
│ PHASE 4 — MODEL                            │
│ Entities + ownership + state + invariants  │
└──────────────────────┬──────────────────────┘
                       ↓
┌─────────────────────────────────────────────┐
│ PHASE 5 — ARCHITECT                        │
│ Boundaries + dependencies + data + errors  │
└──────────────────────┬──────────────────────┘
                       ↓
┌─────────────────────────────────────────────┐
│ PHASE 6 — FOUNDATION                       │
│ Android + network + DB + DI + testing      │
└──────────────────────┬──────────────────────┘
                       ↓
┌─────────────────────────────────────────────┐
│ PHASE 7 — BUILD                            │
│ One complete vertical slice at a time     │
└──────────────────────┬──────────────────────┘
                       ↓
┌─────────────────────────────────────────────┐
│ PHASE 8 — INTEGRATE                        │
│ Complete end-to-end user journeys          │
└──────────────────────┬──────────────────────┘
                       ↓
┌─────────────────────────────────────────────┐
│ PHASE 9 — HARDEN                           │
│ Failure + concurrency + lifecycle          │
│ security + performance + observability     │
└──────────────────────┬──────────────────────┘
                       ↓
┌─────────────────────────────────────────────┐
│ PHASE 10 — RELEASE                         │
│ Reference comparison + acceptance + build  │
└─────────────────────────────────────────────┘
```

---

# 29. Final Rule

When you're confused about what to do next, use this decision tree:

```text
Do I understand the reference behavior?
        │
       NO
        ↓
   Explore it more.

       YES
        ↓
Do I understand the requirement?
        │
       NO
        ↓
   Specify it.

       YES
        ↓
Do I understand the domain/state/ownership?
        │
       NO
        ↓
   Model it.

       YES
        ↓
Do I know where it belongs architecturally?
        │
       NO
        ↓
   Design the boundary.

       YES
        ↓
Can I implement it as a complete vertical slice?
        │
       NO
        ↓
Resolve missing dependencies.

       YES
        ↓
Implement.
        ↓
Test.
        ↓
Break it.
        ↓
Harden it.
        ↓
Mark DONE.
        ↓
Next capability.
```

# 30. The One-Line Workflow

> **Observe → Understand → Specify → Model → Architect → Build → Test → Integrate → Break → Harden → Release.**

Follow that loop for every application.

The biggest change from your previous approach is that **you are no longer developing "screens." You're progressively reconstructing a system, with the screens being one of its outputs.**
