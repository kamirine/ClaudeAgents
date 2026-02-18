---
description: "Run phase delivery gate checks to verify all requirements for a phase are met before proceeding. Validates functional, architectural, performance, and security criteria."
args: "phase=<1|2|3>"
---

# Phase Delivery Gate

Run comprehensive delivery gate checks for a phase to determine readiness for the next phase or production deployment.

## Arguments

- `phase` — Phase number (1, 2, or 3)

## Phase Definitions

### Phase 1: Search & Export
**Steps covered**: 1-5 (infra/entities), 6-7 (search/export/audit services), 8 (SearchController + AuthController), 11-12 (shared libraries), 15-16 (Transaction Explorer + Payment Methods MFEs)

**Gate criteria**:
- 100% unit tests pass
- 100% integration tests pass for Phase 1 endpoints
- Storybook renders for SmartGrid, Transaction Explorer, Payment Methods
- Contract tests pass (OpenAPI snapshot, cursor pagination shape)

### Phase 2: Dashboard & Analytics
**Steps covered**: 6-7 (aggregation/alert/token services), 8 (DashboardController + AlertsController), 9 (SignalR hub), 10 (monitors + snapshots), 13-14 (host app + Dashboard Home), 17-18 (Org Management + Analytics)

**Gate criteria**:
- All Phase 1 + Phase 2 tests pass
- Load tests pass (search + KPI)
- SignalR connection stability verified
- Monitor heartbeats healthy

### Phase 3: Ops Console & AI
**Steps covered**: 6-7 (AI query service), 8 (AiQueryController + AdminController), 19 (Ops Console MFE)

**Gate criteria**:
- All Phase 1 + 2 + 3 tests pass
- Security audit clean
- Performance targets met
- AI prototype go/no-go decided

## Execution Flow

### 1. Inventory Check

Verify all expected files for the phase exist:

```
Phase 1 files:
├── Startup.Dashboard.cs
├── All V2/Dashboard/ API models
├── SavedFilter, ExportJob, AuditLogEntry entities
├── EF configurations for Phase 1 entities
├── Migration 1 (AddDashboardSearchEntities)
├── Search, Export, Audit, FilterPreset services
├── SearchController, AuthController
├── dashboard-shared, smart-grid libraries
├── Transaction Explorer MFE
└── Payment Methods Dashboard MFE

Phase 2 files (in addition to Phase 1):
├── AlertRule, Alert, DashboardKpiSnapshot, RefreshToken entities
├── EF configurations for Phase 2 entities
├── Migration 2 (AddDashboardMonitoringEntities)
├── DashboardAggregation, Alert, Token services
├── DashboardController, AlertsController
├── DashboardHub (SignalR)
├── All 7 background monitors + KpiSnapshotUpdater
├── Host app with routes
├── Dashboard Home, Org Management, Analytics MFEs
└── Notification bell component

Phase 3 files (in addition to Phase 1+2):
├── AI Query service
├── AiQueryController, AdminController
└── Ops Console MFE (monitoring, AI, events, admin)
```

### 2. Architecture Constraint Audit

Run the full 11-constraint check across ALL files in the phase:
1. Startup.cs untouched
2. No ALTER TABLE
3. All entities org-scoped
4. Cursor pagination only
5. Read replica for reads
6. PII masking by default
7. Audit every action
8. Rate limit every endpoint
9. Feature flag everything
10. Redis graceful degradation
11. V1/V2 backward compatibility

### 3. Test Execution

```bash
# Unit tests
dotnet test tests/Unit/Dashboard/ --logger "console;verbosity=detailed"

# Integration tests
dotnet test tests/Integration/Dashboard/ --logger "console;verbosity=detailed"

# Contract tests
dotnet test tests/Integration/Contract/ --logger "console;verbosity=detailed"

# Frontend - Storybook build
cd C:\GIT\Micros\payments-mfe
npm run build-storybook
```

### 4. Security Validation

Cross-reference against Security Validation section of validation-plan.md:
- All V2 endpoints require auth
- All V2 endpoints org-scoped
- All V2 endpoints audit-logged
- Cross-org access blocked
- PII masking enforced
- Rate limiting enforced
- Input validation on all fields

### 5. Performance Baseline (Phase 2+)

Verify performance targets from design-document.md:
- KPI query < 50ms P95
- Transaction search (10 filters) < 500ms P95
- Export 10K rows CSV < 30s
- SignalR message delivery < 1s
- MFE bundle size < 300KB each

### 6. Generate Gate Report

```markdown
# Phase {N} Delivery Gate Report

## Overall Status: {PASS | FAIL | CONDITIONAL PASS}

## Inventory: {X}/{Y} files present ({percent}%)

## Architecture Compliance:
| Constraint | Status | Notes |
|-----------|--------|-------|
| Startup.cs untouched | PASS | |
| No ALTER TABLE | PASS | |
| ... | ... | ... |

## Test Results:
| Suite | Pass | Fail | Skip | Status |
|-------|------|------|------|--------|
| Unit tests | 45 | 0 | 2 | PASS |
| Integration tests | 30 | 2 | 5 | CONDITIONAL |
| Contract tests | 7 | 0 | 0 | PASS |
| Storybook build | - | - | - | PASS |

## Security Audit: {PASS | FAIL}
- {findings}

## Performance: {PASS | FAIL | NOT TESTED}
- {metrics}

## Blocking Issues:
1. {issue}

## Non-Blocking Issues:
1. {issue}

## Recommendation: {PROCEED TO PHASE N+1 | FIX BLOCKING ISSUES FIRST}
```
