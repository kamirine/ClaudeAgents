---
description: "Execute one implementation step from the 19-step implementation plan. Reads the plan, identifies all files for that step, generates code, and runs build verification. Tracks progress across sessions."
args: "step=<step-number>"
---

# Implement Step

Execute a specific step from the implementation plan (`implementation-plan.md`). This skill orchestrates the full implementation of a step including code generation, pattern compliance, and build verification.

## Arguments

- `step` — Step number (1-19) from the implementation plan

## Step Overview

| Step | Description | Phase | Agent |
|------|------------|-------|-------|
| 1 | Infrastructure Prerequisites (Startup.Dashboard.cs) | 1 | backend-impl |
| 2 | API Models in apimodels Submodule | 1 | api-models |
| 3 | Backend Data Entities (Org-Scoped) | 1 | backend-impl |
| 4 | EF Configurations + DbContext | 1 | backend-impl |
| 5 | Database Migrations (2 migrations) | 1 | backend-impl |
| 6 | Service Contracts | 1 | backend-impl |
| 7 | Service Implementations | 1-3 | backend-impl |
| 8 | API Controllers (V2) | 1-3 | backend-impl |
| 9 | SignalR DashboardHub | 2 | backend-impl |
| 10 | Background Monitoring Services | 2 | backend-impl |
| 11 | Regenerate TypeScript Client | 1 | backend-impl |
| 12 | Frontend Shared Libraries | 1 | frontend-impl |
| 13 | Host App Updates | 1 | frontend-impl |
| 14 | Dashboard Home MFE (port 8883) | 2 | frontend-impl |
| 15 | Transaction Explorer MFE (port 8884) | 1 | frontend-impl |
| 16 | Payment Methods Dashboard MFE (port 8885) | 1 | frontend-impl |
| 17 | Org Management MFE (port 8886) | 2 | frontend-impl |
| 18 | Analytics MFE (port 8887) | 2 | frontend-impl |
| 19 | Ops Console MFE (port 8890) | 3 | frontend-impl |

## Execution Flow

### 1. Read the Implementation Plan

Read the specific step from `C:\Users\kamirineni\Documents\Task\implementation-plan.md`:
- What files to create/modify
- What patterns to follow
- What code samples are provided

### 2. Read the Validation Plan

Read the corresponding test cases from `C:\Users\kamirineni\Documents\Task\validation-plan.md`:
- What tests will validate this step
- What assertions must pass

### 3. Explore Existing Patterns

Before writing code, explore the existing codebase to understand:
- How existing similar files are structured
- What naming conventions are used
- What imports/references are needed

### 4. Use Scaffolding Skills

Where applicable, use the scaffolding skills first:
- Step 2: Create model files manually (too specific for scaffold)
- Step 3: Use `/scaffold-ef-config` for each entity
- Step 6-7: Use `/scaffold-service` for each service
- Step 8: Use `/scaffold-v2-controller` for each controller
- Step 10: Use `/scaffold-monitor` for each monitor
- Steps 14-19: Use `/scaffold-mfe` for each MFE package

### 5. Implement the Step

For backend steps (1-11):
1. Create all files specified in the step
2. Follow all architecture constraints from CLAUDE.md
3. Use code samples from implementation-plan.md as reference
4. Ensure org-scoping, audit logging, rate limiting, PII masking

For frontend steps (12-19):
1. Create all files specified in the step
2. Follow MFE patterns from the Frontend Patterns Reference
3. Create Storybook stories with required states
4. Create MSW mock handlers
5. Ensure ARIA labels and keyboard accessibility

### 6. Verify Build

For backend steps:
```bash
cd C:\Users\kamirineni\Documents\Task\modularpayments
dotnet build src/modularpayments/modularpayments.csproj
```

For frontend steps:
```bash
cd C:\GIT\Micros\payments-mfe
npx tsc --noEmit --project packages/{name}/tsconfig.json
```

### 7. Per-Step Completion Checklist

Before marking the step as complete, verify:
- [ ] All files from the step are created/modified
- [ ] No compiler warnings
- [ ] Build succeeds
- [ ] Org-scoping enforced (backend)
- [ ] Audit logging added (backend)
- [ ] Rate limiting applied (backend)
- [ ] Feature flag gating applied where specified
- [ ] Storybook stories render all states (frontend)
- [ ] MSW mocks match API contracts (frontend)
- [ ] Error boundary wraps MFE root (frontend)
- [ ] ARIA labels on interactive elements (frontend)

### 8. Record Progress

After completing the step, report:
- Files created/modified (with paths)
- Architecture constraints verified
- Build status (pass/fail)
- Any issues or deviations from the plan
