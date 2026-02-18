---
description: "Validate a completed implementation step against the validation plan. Runs all test cases for the specified step, checks architecture compliance, and generates a validation report."
args: "step=<step-number>"
---

# Validate Step

Validate a completed implementation step against `validation-plan.md`. This skill checks that all expected test cases can be satisfied and that the implementation follows architecture constraints.

## Arguments

- `step` — Step number (1-19) from the implementation plan

## Execution Flow

### 1. Load Validation Criteria

Read the step's test cases from `C:\Users\kamirineni\Documents\Task\validation-plan.md`:
- Extract all test case names and assertions
- Note the expected behavior for each test

### 2. Load Implementation Plan Requirements

Read the step's requirements from `C:\Users\kamirineni\Documents\Task\implementation-plan.md`:
- What files should exist
- What patterns should be followed

### 3. File Existence Check

Verify all expected files from the step exist:

**Backend file locations**:
- Controllers: `modularpayments/src/modularpayments/Controllers/V2/`
- Services: `modularpayments/src/modularpayments.services/`
- Entities: `modularpayments/src/modularpayments.data/`
- EF Configs: `modularpayments/src/modularpayments.database.postgres/EfConfig/`
- Migrations: `modularpayments/src/modularpayments.database.postgres/Migrations/`
- Background services: `modularpayments/src/modularpayments/BackgroundServices/Dashboard/`
- SignalR: `modularpayments/src/modularpayments/Hubs/`

**Frontend file locations**:
- Libraries: `payments-mfe/libraries/`
- MFE packages: `payments-mfe/packages/`
- Stories: `packages/*/src/**/*.stories.tsx`
- MSW mocks: `packages/*/src/mocks/handlers.ts`

### 4. Architecture Constraint Verification

Check each file against the 11 non-negotiable constraints:

1. **Startup.cs untouched**: Grep for modifications
2. **No ALTER TABLE**: Check migration files for ALTER TABLE statements
3. **Org-scoped**: Verify OrganizationId in queries
4. **Cursor pagination**: Verify CursorPaginatedResponse usage
5. **Read replica**: Verify DashboardReadOnlyDbContext for reads
6. **PII masking**: Check for view:pii permission checks
7. **Audit logging**: Verify IAuditService calls
8. **Rate limiting**: Verify EnableRateLimiting attributes
9. **Feature flags**: Check LaunchDarkly flag checks
10. **Redis fallback**: Verify try/catch around Redis calls
11. **V1 unchanged**: Verify no V1 file modifications

### 5. Code Quality Checks

**Backend**:
- [ ] All async methods have CancellationToken
- [ ] Structured logging (no string interpolation)
- [ ] Constructor injection only
- [ ] Background services use IDbContextFactory
- [ ] ProducesResponseType on all controller actions
- [ ] RequireOrganizationScope on V2 controllers
- [ ] ValidateModelV2 on V2 controllers

**Frontend**:
- [ ] Functional components only
- [ ] ARIA labels on interactive elements
- [ ] ErrorBoundary wraps MFE roots
- [ ] React Query for server state (no Redux)
- [ ] CursorPaginatedResponse shapes in MSW mocks
- [ ] ApiProblemDetails shapes in error mocks
- [ ] Required story states (Default, Loading, Empty, Error)

### 6. Test Case Mapping

For each test case in the validation plan:
- Check if corresponding test file exists
- If test exists, verify it tests the correct assertion
- If test doesn't exist, flag as missing

### 7. Build Verification

```bash
# Backend
cd C:\Users\kamirineni\Documents\Task\modularpayments
dotnet build

# Run unit tests for the step
dotnet test tests/Unit/Dashboard/ --filter "Category={step}"

# Run integration tests for the step
dotnet test tests/Integration/Dashboard/ --filter "Category={step}"
```

### 8. Generate Validation Report

Output a structured report:

```markdown
# Validation Report — Step {N}: {Title}

## File Existence: {PASS|FAIL}
- [x] file1.cs
- [ ] file2.cs (MISSING)

## Architecture Constraints: {PASS|FAIL}
- [x] Startup.cs untouched
- [x] No ALTER TABLE
- [ ] Missing audit logging on {endpoint}

## Test Coverage: {X}/{Y} ({percent}%)
- [x] Test_Name_1 — PASS
- [ ] Test_Name_2 — NOT IMPLEMENTED
- [x] Test_Name_3 — PASS

## Build Status: {PASS|FAIL}
- Backend build: PASS
- Unit tests: 15/15 PASS
- Integration tests: 8/10 PASS (2 pending)

## Issues Found:
1. {issue description}

## Recommendations:
1. {recommendation}
```
