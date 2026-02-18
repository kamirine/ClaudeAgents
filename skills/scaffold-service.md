---
description: "Scaffold a new dashboard service: creates interface in Contracts/Dashboard/ and implementation in Implementation/Dashboard/. Verifies ServicesModule.cs auto-scan covers the new namespace."
args: "name=<ServiceName>"
---

# Scaffold Dashboard Service

Create a service interface and implementation for the ModularPayments dashboard.

## Arguments

- `name` — Service name without "Service" suffix (e.g., `DashboardAggregation`, `Alert`, `ExportJob`, `FilterPreset`, `Audit`, `Token`, `AiQuery`, `Admin`)

## Steps

### 1. Read existing patterns
Read an existing service contract and implementation for reference:
- `C:\Users\kamirineni\Documents\Task\modularpayments\src\modularpayments.services\Contracts\ITransactionService.cs`
- Check `ServicesModule.cs` to understand auto-scan namespace

### 2. Create service interface

File: `src/modularpayments.services/Contracts/Dashboard/I{name}Service.cs`

```csharp
using ServiceTitan.Fintech.ModularPayments.ApiModels.V2.Dashboard;

namespace ServiceTitan.Fintech.ModularPayments.Services.Contracts.Dashboard;

/// <summary>
/// {Description} service for the payments dashboard.
/// All methods require organizationId for multi-tenant scoping.
/// </summary>
public interface I{name}Service
{
    // TODO: Add method signatures per design-document.md
    // Every method MUST:
    //   - Accept Guid organizationId as first parameter
    //   - Accept CancellationToken as last parameter
    //   - Be async (return Task<T>)
}
```

### 3. Create service implementation

File: `src/modularpayments.services/Implementation/Dashboard/{name}Service.cs`

```csharp
using Microsoft.Extensions.Logging;
using ServiceTitan.Fintech.ModularPayments.Services.Contracts.Dashboard;

namespace ServiceTitan.Fintech.ModularPayments.Services.Implementation.Dashboard;

/// <summary>
/// Implementation of I{name}Service.
/// </summary>
public class {name}Service : I{name}Service
{
    private readonly ILogger<{name}Service> _logger;

    public {name}Service(ILogger<{name}Service> logger)
    {
        _logger = logger;
    }

    // TODO: Implement methods
    // Use structured logging:
    //   _logger.LogInformation("Operation completed {@OrganizationId} {@ResultCount}", orgId, count);
    // NOT string interpolation:
    //   _logger.LogInformation($"Operation completed {orgId}"); // BAD
}
```

### 4. Verify auto-scan registration

Read `src/modularpayments.services/ServicesModule.cs` and verify that the `Implementation.Dashboard` namespace is covered by the auto-scan. The existing scan typically uses:
```csharp
.Where(t => t.Namespace != null && t.Namespace.StartsWith(typeof(ManagementService).Namespace!))
```

Since `ManagementService` is in `Implementation`, and `Dashboard` is a sub-namespace of `Implementation`, the auto-scan should cover it automatically. Confirm this.

### 5. Special cases

**For AuditService**: This is split into two classes:
- `AuditService : IAuditService` — producer (writes to bounded channel)
- `AuditBackgroundWriter : BackgroundService` — consumer (reads from channel, writes to DB)
- The `AuditBackgroundWriter` MUST be registered via `AddHostedService` in `Startup.Dashboard.cs`, NOT via auto-scan

**For background services**: If this service is a `BackgroundService` or inherits `MonitorBase`, it must be registered explicitly via `AddHostedService<T>()` in `Startup.Dashboard.cs`. Auto-scan doesn't handle `BackgroundService` subclasses correctly.

### 6. Verify compilation
```bash
dotnet build src/modularpayments.services/modularpayments.services.csproj
```
