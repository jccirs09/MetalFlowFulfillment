# MetalFlow System Roadmap
# EXECUTION BLUEPRINT (AGENT-READY)
# Blazor + MudBlazor + .NET 10

============================================================
SECTION 1 — PURPOSE
============================================================

This document defines the full execution roadmap for building MetalFlow.

It bridges:
- AGENTS.md (what the system does)
- CODEX_RULES.md (how code must be written)

This file tells AI agents:

WHAT to build
WHEN to build it
IN WHAT ORDER

Agents must follow this sequence strictly.

Do NOT skip phases.
Do NOT mix phases.
Do NOT implement future logic early.

============================================================
SECTION 2 — DEVELOPMENT STRATEGY
============================================================

MetalFlow must be built in layers:

1. Foundation (security, branch, users)
2. Configuration (machines, stations, trucks, shifts)
3. Intake (picking list)
4. Fulfillment (production + pick/pack)
5. Logistics (load planning + verification)
6. Delivery + reporting

Each layer depends on the previous.

============================================================
SECTION 3 — PHASE OVERVIEW
============================================================

PHASE A — FOUNDATION (ADMIN + SECURITY)
PHASE B — BRANCH CONFIGURATION
PHASE C — SETUP GATES
PHASE D — PICKING LIST INTAKE
PHASE E — FULFILLMENT (PRODUCTION + PICK & PACK)
PHASE F — LOGISTICS (LOAD PLANNING)
PHASE G — LOAD VERIFICATION
PHASE H — DELIVERY TRACKING
PHASE I — REPORTING

============================================================
PHASE A — FOUNDATION (ADMIN + SECURITY)
============================================================

PURPOSE:
Establish identity, branch structure, and permissions.

MODULES:

1. Branch Management
2. User Management
3. User Branch Membership
4. User Branch Roles

ENTITIES:

Branch
ApplicationUser
UserBranchMembership
UserBranchRole

SERVICES:

IBranchService
IUserAdminService
IUserBranchMembershipService
IUserBranchRoleService

CORE FEATURES:

- Create / Update / Activate / Deactivate Branch
- Create / Update / Activate / Deactivate User
- Assign user to branch
- Set default branch
- Assign roles per branch
- Resolve active branch context
- RoleResolver implementation

RULES:

- Only SystemAdmin manages branches
- Branch roles are DB-based only
- Users cannot operate without active membership
- One default branch per user

OUTPUT:

✔ Secure login  
✔ Branch switching  
✔ Role-based access working  

============================================================
PHASE B — BRANCH CONFIGURATION
============================================================

PURPOSE:
Define operational environment per branch.

MODULES:

1. Truck Management
2. Shift Template Management
3. Pack Station Management
4. Machine Management

ENTITIES:

Truck
ShiftTemplate
PackStation
Machine

SERVICES:

ITruckService
IShiftTemplateService
IPackStationService
IMachineService

CORE FEATURES:

- Create / Update / Deactivate trucks
- Define max weight (critical for load planning)
- Create shift templates (start/end times)
- Define pack stations (tables)
- Define machines (CTL / Slitter)

RULES:

- All data is branch-scoped
- Cannot assign across branches
- Must exist before operations begin

OUTPUT:

✔ Branch fully configured  
✔ Equipment and stations defined  

============================================================
PHASE C — SETUP GATES
============================================================

PURPOSE:
Prevent operations until configuration is complete.

SERVICE:

ISetupStatusService

RULES:

Branch is NOT operational unless:

- ≥1 Machine exists
- ≥1 Pack Station exists
- ≥1 Shift Template exists
- ≥1 Truck exists

FEATURES:

- SetupDashboard UI
- Red/Green status per requirement
- Block all operational modules if incomplete

OUTPUT:

✔ Controlled system activation  
✔ Prevents invalid operation states  

============================================================
PHASE D — PICKING LIST INTAKE
============================================================

PURPOSE:
Planner inputs ERP picking list into MetalFlow.

MODULE:

Picking List Intake

ENTITIES:

PickingListHeader
PickingListLine

SERVICES:

IPickingListService

PROCESS:

Planner receives picking list via email  
→ Planner inputs/imports into system  
→ System validates structure  
→ System stores order  

RULES:

- ERP is source of truth
- No modification of ERP data
- Must support manual entry (no API assumed)
- Data reused across all downstream modules

OUTPUT:

✔ Orders exist in system  
✔ Ready for fulfillment  

============================================================
PHASE E — FULFILLMENT
============================================================

PURPOSE:
Execute production OR pick & pack.

SUBMODULES:

1. Work Orders (Production)
2. Pick & Pack

----------------------------------------
E1 — WORK ORDERS (PRODUCTION)
----------------------------------------

ENTITIES:

WorkOrder
WorkOrderLink
Tag

SERVICES:

IWorkOrderService

RULES:

- Only created when production required
- Tied to picking list
- CTL: 1 skid = 1 tag
- Slitter: 1 coil = 1 tag

----------------------------------------
E2 — PICK & PACK
----------------------------------------

ENTITIES:

PickPackTask
PickPackTaskLine
Tag

SERVICES:

IPickPackService

PROCESS:

Picker pulls material  
→ Packager wraps/bands  
→ Tag assigned to order  

RULES:

- Inventory-based only
- Supports partial tag usage
- Supports multi-tag consolidation
- Tag always required

OUTPUT:

✔ Orders fulfilled  
✔ Tags assigned  

============================================================
PHASE F — LOGISTICS (LOAD PLANNING)
============================================================

PURPOSE:
Plan truck loads.

ENTITIES:

LoadPlan
LoadPlanReservation

SERVICES:

ILoadPlanningService

PROCESS:

Planner groups orders:

- Local delivery (multi-drop)
- Island schedule
- Branch transfer
- Furtherance via Delta

RULES:

- Truck capacity must be respected
- Orders can be reassigned before locking
- Mixed ship dates allowed (transfer scenarios)

OUTPUT:

✔ Truck loads created  
✔ Orders assigned to loads  

============================================================
PHASE G — LOAD VERIFICATION
============================================================

PURPOSE:
Ensure physical load matches plan.

ENTITIES:

LoadScanSession
LoadScan

SERVICES:

ILoadVerificationService

PROCESS:

Loader A loads  
Loader B verifies  

RULES:

- Tag must match packing list exactly
- No fuzzy matching
- Verification is mandatory
- No auto-verification

OUTPUT:

✔ Verified loads  
✔ Ready for shipment  

============================================================
PHASE H — DELIVERY TRACKING
============================================================

PURPOSE:
Track delivery lifecycle.

ENTITIES:

DeliveryRecord (or DeliveryProof)

SERVICES:

IDeliveryService

PROCESS:

Driver records:

- departure time
- arrival time
- delivery completion

RULES:

- Customer pickup tracked separately
- Missed pickup carries forward

OUTPUT:

✔ Delivery traceability  

============================================================
PHASE I — REPORTING
============================================================

PURPOSE:
Operational visibility.

SERVICES:

IReportingService

FEATURES:

- Live queries only
- No data warehouse tables
- CSV export

REPORT TYPES:

- Order status
- Production status
- Load status
- Delivery status

OUTPUT:

✔ Operational insights  

============================================================
SECTION 4 — BUILD ORDER (STRICT)
============================================================

Step 1:
Foundation (Phase A)

Step 2:
Branch Configuration (Phase B)

Step 3:
Setup Gates (Phase C)

Step 4:
Picking List Intake (Phase D)

Step 5:
Fulfillment (Phase E)

Step 6:
Load Planning (Phase F)

Step 7:
Load Verification (Phase G)

Step 8:
Delivery (Phase H)

Step 9:
Reporting (Phase I)

============================================================
SECTION 5 — CRITICAL RULES
============================================================

- Do NOT build fulfillment before admin + config
- Do NOT build logistics before fulfillment
- Do NOT skip setup gates
- Do NOT bypass branch scoping
- Do NOT invent workflow states
- Do NOT implement features outside current phase

============================================================
SECTION 6 — DEFINITION OF SUCCESS
============================================================

System is complete when:

- Planner can input picking list
- Orders flow through production OR pick/pack
- Tags track all material
- Loads are planned and verified
- Deliveries are tracked
- All data is branch-scoped
- No duplicate data entry required

============================================================
END OF ROADMAP
============================================================
