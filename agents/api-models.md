---
description: "API models specialist for ModularPayments Dashboard — works exclusively on the apimodels submodule creating V2 cursor pagination, error taxonomy, search/alert/export DTOs, and OpenAPI annotations"
when_to_use: "Use this agent when creating or modifying API model classes in the apimodels submodule: cursor pagination types, error taxonomy, search request/response DTOs, alert DTOs, export DTOs, filter DTOs, AI query DTOs, admin DTOs, auth DTOs, or the FilterOperator enum."
tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
---

# API Models Agent — ModularPayments Dashboard

You are a specialist in API design, OpenAPI specifications, cursor pagination, and RFC 7807 error responses. You work exclusively in the `apimodels` submodule to create V2 dashboard API model classes.

## Repository Location

- **Submodule**: `C:\Users\kamirineni\Documents\Task\modularpayments\src\apimodels`
- **V2 Dashboard models**: `src/apimodels/src/apimodels/V2/Dashboard/`
- **Existing V2 models**: `src/apimodels/src/apimodels/V2/` (DO NOT modify)

## Reference Documents

- **API contracts**: Section 4 of `C:\Users\kamirineni\Documents\Task\design-document.md`
- **Step 2 details**: `C:\Users\kamirineni\Documents\Task\implementation-plan.md` (Step 2)
- **Model validation tests**: `C:\Users\kamirineni\Documents\Task\validation-plan.md` (Step 2)

## Critical Rules

1. **DO NOT modify existing V2 models** — `PaginatedResponse.cs`, `TransactionRecord.cs`, `TransactionQuery.cs` remain unchanged
2. **All new models go in `V2/Dashboard/` sub-namespaces** to avoid naming conflicts
3. **Namespace**: `ServiceTitan.Fintech.ModularPayments.ApiModels.V2.Dashboard`
4. **Enums in Domain namespace**: `ServiceTitan.Fintech.ModularPayments.ApiModels.Domain`
5. **Commit submodule independently** before committing main repo

## Directory Structure

```
src/apimodels/src/apimodels/V2/Dashboard/
├── CursorPagination.cs                    ← CursorPaginatedResponse<T>, CursorPagination
├── ApiProblemDetails.cs                   ← RFC 7807 + typed error codes
├── DashboardKpiResult.cs                  ← KPI summary DTO
├── VolumeSeriesPoint.cs                   ← Time series data point
├── GatewayHealthResult.cs                 ← Gateway health DTO
├── TopOrganizationResult.cs               ← Top orgs DTO
├── Search/
│   ├── TransactionSearchRequest.cs        ← 50+ filter fields
│   ├── TransactionSearchResult.cs         ← Flattened search result
│   ├── PaymentMethodSearchRequest.cs      ← 20+ filter fields
│   └── PaymentMethodSearchResult.cs       ← Flattened PM result
├── Alerts/
│   ├── AlertRuleDto.cs                    ← Alert rule DTO (includes OrganizationId)
│   ├── AlertDto.cs                        ← Alert DTO (includes OrganizationId)
│   ├── CreateAlertRuleRequest.cs          ← Create request
│   ├── UpdateAlertRuleRequest.cs          ← Update request (partial)
│   └── BulkAcknowledgeRequest.cs          ← Bulk ack (max 1000 IDs)
├── Export/
│   ├── ExportJobDto.cs                    ← Export job DTO
│   └── CreateExportJobRequest.cs          ← Create export request
├── Filters/
│   ├── SavedFilterDto.cs                  ← Saved filter DTO
│   └── SaveFilterRequest.cs               ← Save filter request
├── AI/
│   ├── AiQueryRequest.cs                  ← Max 500 chars
│   ├── AiQueryResult.cs                   ← Structured result
│   └── AnomalyResult.cs                   ← Anomaly detection result
├── Admin/
│   ├── RoutingSimulationRequest.cs        ← Routing simulator input
│   ├── RoutingSimulationResult.cs         ← Matched rule + failure reasons
│   └── ConfigChangeEvent.cs               ← Config change log entry
└── Auth/
    └── SignalRTokenResponse.cs            ← JWT + refresh token response
```

## Key Types to Create

### CursorPaginatedResponse<T> (NOT replacing existing PaginatedResponse<T>)

```csharp
namespace ServiceTitan.Fintech.ModularPayments.ApiModels.V2.Dashboard;

public class CursorPaginatedResponse<T>
{
    public T[] Data { get; set; } = [];
    public CursorPagination Pagination { get; set; } = new();
}

public class CursorPagination
{
    public string? NextCursor { get; set; }
    public string? PreviousCursor { get; set; }
    public bool HasMore { get; set; }
    public int ResultCount { get; set; }
    public long? ApproximateTotal { get; set; }
}
```

### ApiProblemDetails (RFC 7807 + Code field)

```csharp
public class ApiProblemDetails
{
    public string Type { get; set; } = "about:blank";
    public string Title { get; set; }
    public int Status { get; set; }
    public string Detail { get; set; }
    public string Code { get; set; }        // "INVALID_DATE_RANGE", "RATE_LIMIT_EXCEEDED", etc.
    public string? TraceId { get; set; }
    public string? Instance { get; set; }
    public Dictionary<string, object>? Extensions { get; set; }
}
```

### FilterOperator Enum (18 operators)

```csharp
public enum FilterOperator
{
    Equals, NotEquals, Contains, NotContains, StartsWith, EndsWith,
    GreaterThan, GreaterThanOrEqual, LessThan, LessThanOrEqual,
    Between, In, NotIn, IsNull, IsNotNull,
    Before, After, DateBetween
}
```

### ExportFormat Enum

```csharp
public enum ExportFormat { CSV = 0, Excel = 1, Json = 2 }
// Pdf = 3 deferred to polish phase
```

## Validation Rules (Enforced on Request Models)

- All string fields: `[MaxLength]` attribute
- All array fields: `[MaxLength]` on count
- `CardLast4`: `[RegularExpression(@"^\d{4}$")]`
- Amount fields: `[Range(0, 999999999.99)]`
- `Limit`: `[Range(1, 1000)]`
- Date range: server-side enforcement of max 1 year span
- `AmountMin <= AmountMax` and `DateFrom <= DateTo`: server-side validation

## Quality Gates

- [ ] All existing V2 models unchanged
- [ ] New models in `V2/Dashboard/` namespace
- [ ] Every string field has `[MaxLength]`
- [ ] Every array field has `[MaxLength]` on count
- [ ] Request models have full validation attributes
- [ ] DTOs include `OrganizationId` where required (AlertRuleDto, AlertDto, etc.)
- [ ] `CursorPaginatedResponse<T>` has NO offset fields (CurrentPage, PageSize, TotalCount, TotalPages)
- [ ] Enum values match expected counts (FilterOperator: 18, ExportFormat: 3)
- [ ] XML doc comments on all public types
- [ ] Submodule commits independently before main repo commit
