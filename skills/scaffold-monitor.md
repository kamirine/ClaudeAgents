---
description: "Scaffold a background monitoring service that inherits MonitorBase. Includes circuit breaker (3 failures → 5 min pause), staggered start, 30s query timeout, heartbeat recording, IDbContextFactory usage, and LaunchDarkly flag gating."
args: "name=<MonitorName> interval=<seconds>"
---

# Scaffold Background Monitor

Create a background monitoring service for the ModularPayments dashboard.

## Arguments

- `name` — Monitor name without "Monitor" suffix (e.g., `GatewayHealth`, `TransactionFailure`, `StaleTransaction`, `PaymentMethodExpiration`, `CaptureReconciliation`, `Latency`)
- `interval` — Check interval in seconds (e.g., `30`, `60`, `300`)

## Monitor Registry

| Monitor | Interval | Alert Condition |
|---------|----------|----------------|
| GatewayHealth | 30s | Success rate < 95% |
| TransactionFailure | 60s | Failure rate > 5% |
| StaleTransaction | 300s | Pending > 30 min |
| PaymentMethodExpiration | 3600s | Expiring cards summary |
| CaptureReconciliation | 300s | High retry count |
| Latency | 60s | P95 > 2x baseline |
| **KpiSnapshotUpdater** | **300s** | **Writes snapshots (not alerts)** |

## Steps

### 1. Read MonitorBase pattern
Check if MonitorBase already exists. If not, create it first:
```
src/modularpayments/BackgroundServices/Dashboard/MonitorBase.cs
```

### 2. Create MonitorBase (if it doesn't exist)

```csharp
public abstract class MonitorBase : BackgroundService
{
    private readonly IDbContextFactory<DashboardReadOnlyDbContext> _contextFactory;
    private readonly ILogger _logger;
    private readonly TimeSpan _interval;
    private int _consecutiveFailures;
    private const int CircuitBreakerThreshold = 3;
    private static readonly TimeSpan CircuitRecoveryTime = TimeSpan.FromMinutes(5);
    private DateTime _circuitOpenedAt = DateTime.MinValue;
    private DateTime _lastHeartbeat;

    protected MonitorBase(
        IDbContextFactory<DashboardReadOnlyDbContext> contextFactory,
        ILogger logger,
        TimeSpan interval)
    {
        _contextFactory = contextFactory;
        _logger = logger;
        _interval = interval;
    }

    protected abstract Task ExecuteCheckAsync(DashboardReadOnlyDbContext context, CancellationToken ct);

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        // Staggered start: hash-based delay
        var stagger = Math.Abs(GetType().Name.GetHashCode()) % 10000;
        await Task.Delay(stagger, stoppingToken);

        while (!stoppingToken.IsCancellationRequested)
        {
            try
            {
                // Circuit breaker check
                if (_consecutiveFailures >= CircuitBreakerThreshold)
                {
                    if (DateTime.UtcNow - _circuitOpenedAt < CircuitRecoveryTime)
                    {
                        await Task.Delay(_interval, stoppingToken);
                        continue;
                    }
                    _consecutiveFailures = 0; // Reset for recovery attempt
                }

                // TODO: Check LaunchDarkly flag 'dashboard-monitors-enabled'

                await using var context = await _contextFactory.CreateDbContextAsync(stoppingToken);
                using var cts = CancellationTokenSource.CreateLinkedTokenSource(stoppingToken);
                cts.CancelAfter(TimeSpan.FromSeconds(30)); // 30s query timeout

                await ExecuteCheckAsync(context, cts.Token);

                _consecutiveFailures = 0;
                _lastHeartbeat = DateTime.UtcNow;
            }
            catch (OperationCanceledException) when (stoppingToken.IsCancellationRequested)
            {
                break;
            }
            catch (Exception ex)
            {
                _consecutiveFailures++;
                if (_consecutiveFailures >= CircuitBreakerThreshold)
                    _circuitOpenedAt = DateTime.UtcNow;

                _logger.LogError(ex, "Monitor {MonitorName} check failed (failure #{Count})",
                    GetType().Name, _consecutiveFailures);
            }

            await Task.Delay(_interval, stoppingToken);
        }
    }

    public DateTime LastHeartbeat => _lastHeartbeat;
    public int ConsecutiveFailures => _consecutiveFailures;
    public bool IsCircuitOpen => _consecutiveFailures >= CircuitBreakerThreshold;
}
```

### 3. Create the specific monitor

File: `src/modularpayments/BackgroundServices/Dashboard/{name}Monitor.cs`

```csharp
public class {name}Monitor : MonitorBase
{
    private readonly IAlertService _alertService;
    private readonly IHubContext<DashboardHub> _hubContext;

    public {name}Monitor(
        IDbContextFactory<DashboardReadOnlyDbContext> contextFactory,
        IAlertService alertService,
        IHubContext<DashboardHub> hubContext,
        ILogger<{name}Monitor> logger)
        : base(contextFactory, logger, TimeSpan.FromSeconds({interval}))
    {
        _alertService = alertService;
        _hubContext = hubContext;
    }

    protected override async Task ExecuteCheckAsync(DashboardReadOnlyDbContext context, CancellationToken ct)
    {
        // TODO: Implement check logic
        // 1. Query the read replica for relevant metrics
        // 2. Compare against thresholds
        // 3. Create/update alerts via _alertService
        // 4. Push real-time updates via _hubContext
    }
}
```

### 4. Register in Startup.Dashboard.cs

Add to `Startup.Dashboard.cs`:
```csharp
services.AddHostedService<{name}Monitor>();
```

**Important**: Background services MUST be registered via `AddHostedService`, NOT via Autofac auto-scan.

### 5. Verify
- Monitor compiles
- Uses `IDbContextFactory<DashboardReadOnlyDbContext>` (NOT injected DbContext)
- Has circuit breaker (3 failures → 5 min pause)
- Has 30s query timeout
- Has staggered start delay
- Records heartbeat timestamp
- Registered in Startup.Dashboard.cs
