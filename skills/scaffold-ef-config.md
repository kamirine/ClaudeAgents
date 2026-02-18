---
description: "Scaffold an EF Core entity configuration class for a dashboard entity. Creates the configuration following existing FK conventions, index naming, column types, and snake_case naming."
args: "entity=<EntityName>"
---

# Scaffold EF Entity Configuration

Create an Entity Framework Core configuration class for a dashboard entity.

## Arguments

- `entity` — Entity class name (e.g., `AlertRule`, `Alert`, `SavedFilter`, `ExportJob`, `AuditLogEntry`, `DashboardKpiSnapshot`, `RefreshToken`)

## Steps

### 1. Read existing patterns
Read an existing EF configuration for reference:
```
C:\Users\kamirineni\Documents\Task\modularpayments\src\modularpayments.database.postgres\EfConfig\
```
Look at any existing configuration to understand FK conventions, index naming, column types.

Also read the entity class itself to understand its properties and relationships.

### 2. Create configuration file

File: `src/modularpayments.database.postgres/EfConfig/Dashboard/{entity}Configuration.cs`

```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;
using ServiceTitan.Fintech.ModularPayments.Domain;

namespace ServiceTitan.Fintech.ModularPayments.Database.Postgres.EfConfig;

public class {entity}Configuration : IEntityTypeConfiguration<{entity}>
{
    public void Configure(EntityTypeBuilder<{entity}> builder)
    {
        // Table name (snake_case)
        builder.ToTable("{snake_case_table_name}");

        // Primary key (inherited from BaseEntity)
        builder.HasKey(e => e.Id);

        // Foreign keys
        // builder.HasOne(e => e.Organization)
        //     .WithMany()
        //     .HasForeignKey(e => e.OrganizationId)
        //     .OnDelete(DeleteBehavior.Restrict);

        // Indexes
        // builder.HasIndex(e => new { e.OrganizationId, e.Status })
        //     .HasDatabaseName("ix_{table}_{columns}");

        // Column configurations
        // builder.Property(e => e.Name).HasMaxLength(200);
    }
}
```

### 3. Entity-specific configurations

**AlertRule**: FK to Organization, index on (OrganizationId, IsEnabled)
**Alert**: FK to AlertRule + Organization, indexes on (OrganizationId, Status) and (RuleId, TriggeredAt)
**SavedFilter**: FK to Organization, index on (UserId, GridId, OrganizationId)
**ExportJob**: FK to Organization, index on (UserId, Status)
**AuditLogEntry**: Index on (OrganizationId, Action, CreatedOn), NO delete cascade
**DashboardKpiSnapshot**: UNIQUE index on (OrganizationId, SnapshotHour)
**RefreshToken**: Index on HashedToken (NO Organization FK — user-scoped)

### 4. Add DbSet to ModularPaymentsDbContext

Edit `src/modularpayments.database.postgres/ModularPaymentsDbContext.cs` to add:
```csharp
public DbSet<{entity}> {PluralName} { get; set; }
```

### 5. Verify
- Configuration file compiles
- BaseDbContext's `ApplyConfigurationsFromAssembly` will auto-discover this config
- snake_case column names are handled by BaseDbContext's OnConfiguring
- Index names follow `ix_{table}_{column1}_{column2}` convention
- FK delete behavior is Restrict (not Cascade) for dashboard entities
