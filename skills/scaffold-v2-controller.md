---
description: "Scaffold a new V2 API controller with all standard dashboard patterns: RequireOrganizationScope, ValidateModelV2, rate limiting, audit logging, ApiProblemDetails error handling, CursorPaginatedResponse returns."
args: "name=<ControllerName> resource=<route-resource> rateLimit=<rate-limit-policy>"
---

# Scaffold V2 API Controller

Create a new V2 controller at `C:\Users\kamirineni\Documents\Task\modularpayments\src\modularpayments\Controllers\V2\{name}Controller.cs` following all dashboard patterns.

## Arguments

- `name` — Controller name without "Controller" suffix (e.g., `Dashboard`, `Search`, `Alerts`)
- `resource` — Route resource path (e.g., `dashboard`, `search`, `alerts`)
- `rateLimit` — Default rate limit policy (e.g., `kpi-queries`, `transaction-search`)

## Steps

### 1. Read existing patterns
Read the existing V2 TransactionsController to understand the base controller pattern:
`src/modularpayments/Controllers/V2/Services/TransactionsController.cs`

Also read a V1 controller for the `Auth` property pattern:
`src/modularpayments/Controllers/V1/TransactionsController.cs`

### 2. Create the controller file

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.RateLimiting;
using ServiceTitan.Fintech.ModularPayments.ApiModels.V2.Dashboard;
using ServiceTitan.Fintech.ModularPayments.Services.Contracts.Dashboard;

namespace ServiceTitan.Fintech.ModularPayments.Controllers.V2;

/// <summary>
/// {Description} — V2 Dashboard API
/// </summary>
[ApiController]
[Route("publicapi/v2/{resource}")]
[RequireOrganizationScope]
[ApiAuthorize]
[ValidateModelV2]
[ProducesResponseType(typeof(ApiProblemDetails), StatusCodes.Status400BadRequest)]
[ProducesResponseType(typeof(ApiProblemDetails), StatusCodes.Status403Forbidden)]
[ProducesResponseType(typeof(ApiProblemDetails), StatusCodes.Status429TooManyRequests)]
public class {name}Controller : ControllerBase
{
    private readonly I{Service}Service _{service};
    private readonly IAuditService _auditService;
    private readonly ILogger<{name}Controller> _logger;

    public {name}Controller(
        I{Service}Service {service},
        IAuditService auditService,
        ILogger<{name}Controller> logger)
    {
        _{service} = {service};
        _auditService = auditService;
        _logger = logger;
    }

    // Organization ID from auth context
    private Guid OrganizationId => Auth.OrganizationId!.Value;

    // TODO: Add action methods following these patterns:
    //
    // For list endpoints:
    //   [HttpGet, EnableRateLimiting("{rateLimit}")]
    //   [ProducesResponseType(typeof(CursorPaginatedResponse<TDto>), 200)]
    //   public async Task<IActionResult> ListAsync([FromQuery] request, CancellationToken ct)
    //
    // For create endpoints:
    //   [HttpPost, EnableRateLimiting("{rateLimit}")]
    //   [ProducesResponseType(typeof(TDto), 201)]
    //   public async Task<IActionResult> CreateAsync([FromBody] request, CancellationToken ct)
    //
    // Always:
    //   - Use OrganizationId property for org scoping
    //   - Audit log every action via _auditService
    //   - Return ApiProblemDetails for known errors
    //   - Include CancellationToken parameter
}
```

### 3. Key patterns to follow

**Org scoping**: Use `Auth.OrganizationId!.Value` directly. The `[RequireOrganizationScope]` attribute verifies the claim exists.

**Cross-org protection for arrays**:
```csharp
if (request.OrganizationIds?.Any(id => id != OrganizationId) == true)
    return StatusCode(403, new ApiProblemDetails
    {
        Code = "CROSS_ORG_ACCESS_DENIED",
        Title = "Cross-organization access denied",
        Status = 403
    });
```

**Audit logging**:
```csharp
await _auditService.LogSearchAsync(OrganizationId, Auth.UserId, "Transaction",
    request, result.Pagination.ResultCount, (int)sw.ElapsedMilliseconds, HttpContext.Connection.RemoteIpAddress?.ToString());
```

**Error response**:
```csharp
return StatusCode(400, new ApiProblemDetails
{
    Code = "INVALID_DATE_RANGE",
    Title = "Invalid date range",
    Detail = "FromDate must be before ToDate",
    Status = 400,
    TraceId = HttpContext.TraceIdentifier
});
```

### 4. Verify
- Controller compiles: `dotnet build src/modularpayments/modularpayments.csproj`
- All action methods have `[ProducesResponseType]` for success and error codes
- All action methods have `[EnableRateLimiting]`
- `[RequireOrganizationScope]` is on the controller class
- `[ValidateModelV2]` is on the controller class (NOT the V1 `[ValidateModel]`)
