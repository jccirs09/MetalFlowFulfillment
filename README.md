# CODEX_RULES.md
# MetalFlow AI Coding Rules
# Blazor + MudBlazor + .NET 10
# STRICT IMPLEMENTATION DIRECTIVES

============================================================
SECTION 1 — PURPOSE
============================================================

This file defines the coding behavior rules for Codex and any AI agent
working inside the MetalFlow repository.

This file does NOT define business workflow.
Business workflow is defined in AGENTS.md.

This file defines HOW code must be written.

Purpose:

- keep architecture clean
- prevent UI-layer business logic
- prevent branch-scope violations
- prevent bad EF patterns
- prevent fake scaffolding
- enforce maintainable enterprise code

If AGENTS.md defines WHAT the system does,
this file defines HOW the implementation must be done.

============================================================
SECTION 2 — GENERAL AI BEHAVIOR RULES
============================================================

AI agents MUST:

- follow existing project architecture
- prefer extending current patterns over inventing new ones
- generate production-ready code
- keep code explicit and readable
- avoid over-abstraction unless already established in repo
- preserve real-world terminology from AGENTS.md
- ask before making destructive changes

AI agents MUST NOT:

- invent requirements
- rename domain concepts
- create fantasy abstractions
- scaffold placeholder code with TODO logic and pretend it is done
- silently change existing behavior
- add libraries without clear justification
- add JavaScript for behavior that Blazor/MudBlazor can handle directly
- create code that bypasses service layer

============================================================
SECTION 3 — REQUIRED IMPLEMENTATION FLOW
============================================================

Allowed application flow:

UI Component
→ Application Service
→ DbContext
→ Database

Allowed read/write flow:

Razor Component
→ injected service interface
→ service implementation
→ DbContext
→ entities / DTO mapping
→ response back to UI

FORBIDDEN:

Razor Component
→ DbContext

Razor Component
→ direct entity mutation

Razor Component
→ raw SQL

Razor Component
→ business rules

Controllers
→ business rules

Controllers
→ DbContext directly unless controller is only acting as thin transport
for external integration and still defers logic to services

============================================================
SECTION 4 — SOLUTION STRUCTURE RULES
============================================================

Preferred structure:

MetalFlow.Web
- Pages
- Layouts
- Components
- Dialogs
- State
- Auth

MetalFlow.Application
- Interfaces
- Services
- DTOs
- Commands
- Queries
- Validators

MetalFlow.Domain
- Entities
- Enums
- Constants
- ValueObjects (only if clearly justified)

MetalFlow.Infrastructure
- Data
- Configurations
- Migrations
- Identity
- Persistence helpers

AI must place code in the correct project.

Do NOT put EF models/configuration in Web.
Do NOT put UI DTOs in Domain.
Do NOT put MudBlazor UI code in Application.

============================================================
SECTION 5 — BLAZOR RULES
============================================================

Razor components must be thin.

Razor component responsibilities:

- render UI
- capture input
- invoke services
- manage UI state
- show validation/errors
- open dialogs/snackbars
- handle loading states

Razor components must NOT:

- contain branch authorization logic
- contain fulfillment rules
- contain scheduling rules
- contain tag-processing rules
- contain import parsing rules
- contain load-planning rules
- contain database query composition

Code-behind files or partial classes are preferred when component logic
becomes non-trivial.

If a component exceeds reasonable complexity, split into:

- page
- reusable child components
- dialogs
- service calls

============================================================
SECTION 6 — MUD BLAZOR RULES
============================================================

Prefer MudBlazor components before custom HTML.

Use MudBlazor consistently for:

- forms
- dialogs
- tables
- tabs
- drawers
- alerts
- snackbars
- date/time inputs

Do NOT mix multiple UI systems.

Use dense layouts where operational screens need speed.

Warehouse screens must prioritize:

- keyboard flow
- scanner usability
- low click count
- readable status signals

Use full-screen dialogs for hard-stop operational failures when required.

============================================================
SECTION 7 — SERVICE LAYER RULES
============================================================

Every business action must go through a service.

Examples:

- create picking list intake record
- create work order
- assign tag to order
- mark pack complete
- create truck load
- verify truck load
- record driver departure
- update scheduling entry

Services must:

- validate branch context
- validate user permissions where applicable
- enforce business rules
- write audit events where required
- use UTC timestamps
- return clear success/error results

Services should be named clearly.

Good:

- PickingListIntakeService
- WorkOrderService
- TagAssignmentService
- LoadPlanningService
- LoadVerificationService

Bad:

- CommonService
- UtilityService
- DataManager
- LogicHelper

============================================================
SECTION 8 — DTO RULES
============================================================

Use DTOs between UI and services.

Do NOT bind Razor components directly to entities for complex workflows.

Use:

- Create DTOs
- Update DTOs
- View DTOs
- Result DTOs

Entities remain persistence/domain objects.

DTO names should be explicit.

Good:

- CreateWorkOrderRequest
- PickingListSummaryDto
- LoadVerificationResultDto

Bad:

- DataModel
- RequestModel
- ItemData

============================================================
SECTION 9 — DATABASE / EF CORE RULES
============================================================

EF Core Migrations are mandatory.

NEVER use:

EnsureCreated()

Startup must use migrations.

All entity configuration must be explicit.

Use IEntityTypeConfiguration<T> where practical.

Configure:

- required fields
- max lengths
- indexes
- unique constraints
- foreign keys
- concurrency tokens
- delete behavior

Do NOT rely on accidental EF conventions for critical constraints.

============================================================
SECTION 10 — QUERY RULES
============================================================

Queries must be:

- branch-scoped
- explicit
- efficient
- readable

All operational queries MUST filter by active BranchId.

If a service reads operational data and does not filter by branch,
that is a defect.

Prefer projection to DTOs for list/detail screens.

Do NOT eagerly load large object graphs without reason.

Do NOT use lazy-loading magic.

Avoid N+1 query patterns.

For read screens:

- AsNoTracking() by default unless tracked update flow is required

For large tables:

- filtering first
- sorting explicit
- paging explicit

============================================================
SECTION 11 — BRANCH CONTEXT RULES
============================================================

Branch context is mandatory.

Every operational service must resolve and use active branch context.

Do NOT trust UI-selected branch values on their own.

Server-side validation is mandatory.

If a user requests an invalid branch:

- do not switch branch automatically
- return appropriate failure
- preserve current valid context

Never write code that allows cross-branch data leakage.

============================================================
SECTION 12 — AUTHORIZATION RULES
============================================================

Use the project authorization model exactly.

Branch roles must come from database resolution.

Do NOT put branch roles in claims.

Use role resolver / authorization service patterns already defined.

Do NOT scatter inline string role checks everywhere in UI.

Prefer centralized policy checks or service-level authorization checks.

============================================================
SECTION 13 — AUDIT RULES
============================================================

Audit is not optional for critical actions.

When business rules require audit logging, AI must implement it.

Critical operational actions should be auditable, including but not limited to:

- work order creation
- work order updates
- tag assignment / reassignment
- shipment verification
- load verification completion
- override actions
- branch-admin changes

Audit records are append-only.

Never generate code that updates or deletes audit history.

============================================================
SECTION 14 — CONCURRENCY RULES
============================================================

Where concurrency tokens are required, AI must preserve them.

Use optimistic concurrency properly.

Never strip Version columns just because they are inconvenient.

On concurrency conflict:

- service returns structured failure
- UI shows blocking refresh-required message
- do not silently overwrite another user's work

No last-write-wins shortcuts unless explicitly approved.

============================================================
SECTION 15 — SOFT DELETE RULES
============================================================

Operational data is not hard deleted.

Use inactive/status-based handling according to domain rules.

If removal is needed:

- deactivate
- cancel
- close
- supersede

Never permanently delete operational records unless explicitly approved
and clearly non-operational.

============================================================
SECTION 16 — VALIDATION RULES
============================================================

Validation belongs in services and/or validators, not only UI.

UI validation is for user feedback.
Service validation is the real enforcement.

Always validate:

- required fields
- branch scope
- entity existence
- status transitions
- duplicate conflicts
- role permissions
- date/time rules where applicable

Do not trust client input.

============================================================
SECTION 17 — ERROR HANDLING RULES
============================================================

Errors must be explicit and useful.

Do NOT swallow exceptions silently.
Do NOT return vague "Something went wrong" results from services.

Use structured result patterns where already present or appropriate.

Differentiate:

- validation errors
- not found
- forbidden
- concurrency conflict
- integration/import failures
- unexpected system errors

Operational hard-stop errors must be represented clearly in UI.

============================================================
SECTION 18 — IMPORT / PARSING RULES
============================================================

MetalFlow currently operates without direct ERP API connection.

Import/parsing logic must assume:

- planner-driven intake
- imperfect source formatting
- manual review is part of process

Do NOT fake "smart" parsing if accuracy is uncertain.

Import workflows must prefer:

- explicit validation
- visible review
- correction before commit
- clear failure messages

Never auto-correct source data silently.

============================================================
SECTION 19 — TAG RULES
============================================================

Tag handling is core logic.

AI must not simplify tag logic into generic inventory labels.

A tag is an operational tracking unit.

Do NOT create code that treats tags as optional.

Tag-related code must be deterministic and traceable.

Where tag matching is required, do not allow fuzzy matching.

Match exact tag values.

============================================================
SECTION 20 — LOAD / VERIFICATION RULES
============================================================

Load verification is a critical control point.

AI must not weaken it.

Do NOT auto-verify.
Do NOT infer verified tags from planned tags.
Do NOT mark loads complete without verification flow.

Verification-related services should be explicit and auditable.

============================================================
SECTION 21 — STATE / STATUS RULES
============================================================

Do NOT invent extra statuses casually.

Statuses must reflect actual operation.

If introducing a status is necessary, it must be:

- clearly justified
- consistent with AGENTS.md
- minimal
- documented

Avoid bloated workflow state machines unless explicitly required.

============================================================
SECTION 22 — TIME RULES
============================================================

Store all timestamps in UTC.

Convert to branch-local time only in presentation.

Do NOT store local machine times as source of truth.

For scheduling views, make timezone handling explicit.

============================================================
SECTION 23 — NAMING RULES
============================================================

Use real business language from AGENTS.md.

Approved terms include:

- Picking List
- Pack Station
- Ready
- Truck Load
- Shipment
- Tag

Do NOT replace with generic terms like:

- Order Ticket
- Packing Pod
- Dispatch Unit
- Package Label Object

Keep names boring and correct.

============================================================
SECTION 24 — CODE STYLE RULES
============================================================

Generate clean, boring, maintainable code.

Prefer:

- small methods
- explicit naming
- clear control flow
- early validation
- low surprise

Avoid:

- giant god classes
- deeply nested logic
- magical extension-method mazes
- premature generic frameworks
- reflection-heavy tricks
- unnecessary design patterns

Comment only where logic is non-obvious.
Do not narrate obvious code.

============================================================
SECTION 25 — TESTABILITY RULES
============================================================

Write services in a way that is testable.

Prefer constructor injection.
Keep side effects contained.
Separate orchestration from persistence where practical.

Do not bury business rules in UI event handlers.

If adding tests is within scope, prioritize:

- service tests
- validation tests
- branch-scoping tests
- concurrency tests

============================================================
SECTION 26 — PERFORMANCE RULES
============================================================

Operational screens must remain responsive.

Avoid:

- loading entire tables when summary data is enough
- unnecessary joins
- repeated queries in render loops
- chatty service calls from UI

Batch related work where sensible.

Optimize for real branch usage, not theoretical elegance.

============================================================
SECTION 27 — FILE CHANGE RULES
============================================================

Before editing:

- inspect existing patterns
- preserve naming conventions
- preserve folder structure
- avoid unrelated reformatting

Do NOT refactor the whole repo just because you touched one file.

Keep changes scoped to the task.

============================================================
SECTION 28 — WHEN AI SHOULD STOP
============================================================

AI must stop and ask for clarification if:

- a requested change conflicts with AGENTS.md
- workflow meaning is ambiguous
- terminology conflicts with shop-floor meaning
- a destructive change could lose operational data
- branch behavior is uncertain
- existing code suggests two competing patterns and neither is clearly standard

Do NOT guess on operational workflow.

============================================================
SECTION 29 — DEFINITION OF GOOD AI OUTPUT
============================================================

Good AI output for MetalFlow means:

- follows architecture
- respects real workflow
- keeps components thin
- keeps services responsible for logic
- preserves branch boundaries
- preserves auditability
- uses exact business terms
- avoids fake abstractions
- is ready to compile and maintain

If the code looks clever but makes the real process harder to trust,
it is bad output.

============================================================
SECTION 30 — FINAL RULE
============================================================

MetalFlow is an operations system, not a demo app.

Optimize for:

- correctness
- traceability
- maintainability
- real-world branch use

Do not optimize for flashy architecture, trendy patterns,
or AI-generated busywork.

============================================================
END OF CODEX RULES
============================================================
