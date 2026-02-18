---
description: "Scaffold a new MFE (Micro Frontend) package with full boilerplate: package.json, App root, Storybook stories, MSW mocks, API client, Zod schemas. Based on existing packages/transactions/ template."
args: "name=<package-name> port=<port-number> webComponent=<web-component-tag>"
---

# Scaffold MFE Package

Create a complete MFE package directory at `C:\GIT\Micros\payments-mfe\packages/{name}/` using the existing `packages/transactions/` as a template.

## Arguments

- `name` — Package directory name (e.g., `dashboard-home`, `transaction-explorer`)
- `port` — Development server port (e.g., `8883`)
- `webComponent` — Web component tag name (e.g., `payments-dashboard-home`)

## Port Registry (DO NOT reuse existing ports)

| Port | Package |
|------|---------|
| 8881 | payments-transactions (existing) |
| 8882 | payments-transactions-mfe (existing) |
| 8883 | dashboard-home (NEW) |
| 8884 | transaction-explorer (NEW) |
| 8885 | payment-methods-dashboard (NEW) |
| 8886 | org-management (NEW) |
| 8887 | analytics (NEW) |
| 8888 | payments-mfe placeholder (existing) |
| 8889 | payments-pay-mfe (existing) |
| 8890 | ops-console (NEW) |

## Steps

### 1. Read the existing template
Read `C:\GIT\Micros\payments-mfe\packages\transactions\package.json` to understand the exact structure needed.

### 2. Create package.json
```json
{
    "name": "@servicetitan/payments-{name}-mfe",
    "version": "0.0.0-dev.1",
    "dependencies": {
        "@servicetitan/anvil2": "^1.16.6",
        "@servicetitan/modularpayments-client": "^1.0.31",
        "@servicetitan/react-ioc": "^33.0.0",
        "@servicetitan/web-components": "^33.0.0",
        "@tanstack/react-query": "^5.90.12",
        "mobx": "~6.13.5",
        "react": "^18.3.1",
        "react-dom": "^18.3.1"
    },
    "cli": {
        "web-component": {
            "branches": { "qa": { "publishTag": "qa" }, "dev": { "publishTag": "dev" }, "master": { "publishTag": "prod" } }
        },
        "webpack": {
            "port": {port},
            "proxy": "../../local/proxy-settings.js"
        }
    },
    "publishConfig": { "access": "restricted", "registry": "https://verdaccio.servicetitan.com" },
    "files": ["dist/bundle", "dist/metadata.json", "package.json"]
}
```

**NOTE**: No webpack.config.js needed — `cli.webpack` in package.json IS the config.

### 3. Create tsconfig.json
Extends the monorepo root tsconfig.

### 4. Create src/app.tsx
```typescript
import { AnvilProvider } from '@servicetitan/anvil2';
import { FC, StrictMode } from 'react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ErrorBoundary } from '@servicetitan/dashboard-shared';

const queryClient = new QueryClient({
    defaultOptions: { queries: { staleTime: 10000 } },
});

export const App: FC = () => (
    <StrictMode>
        <QueryClientProvider client={queryClient}>
            <AnvilProvider>
                <ErrorBoundary>
                    {/* TODO: Add MFE content */}
                    <div>Dashboard {name} MFE</div>
                </ErrorBoundary>
            </AnvilProvider>
        </QueryClientProvider>
    </StrictMode>
);
```

### 5. Create src/lib/payments-service-client.ts
Copy from `packages/transactions/src/lib/payments-service-client.ts` — uses `getValueForEnvironment()` for base URLs.

### 6. Create src/lib/request.ts
Copy from `packages/transactions/src/lib/request.ts` — custom fetch wrapper with JWT auth.

### 7. Create src/mocks/handlers.ts
```typescript
import { http, HttpResponse } from 'msw';

// Default mock handlers for this MFE's V2 endpoints
export const handlers = [
    // TODO: Add endpoint-specific handlers
];
```

### 8. Create src/app.stories.tsx (CSF3)
```typescript
import type { Meta, StoryObj } from '@storybook/react';
import { App } from './app';
import { handlers } from './mocks/handlers';

const meta = {
    title: 'MFEs/{PascalCaseName}',
    component: App,
    parameters: { layout: 'fullscreen', msw: { handlers } },
    tags: ['autodocs'],
} satisfies Meta<typeof App>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Default: Story = { args: {} };
export const LoadingState: Story = { /* delayed MSW response */ };
export const Empty: Story = { /* 0 results MSW response */ };
export const ErrorState: Story = { /* 429/500 MSW response */ };
```

### 9. Create src/app.webcomponent.stories.tsx
```typescript
import type { Meta, StoryObj } from '@storybook/react';
import { Loader } from '@servicetitan/web-components';

const meta = {
    title: 'WebComponents/{PascalCaseName}',
    component: Loader,
    parameters: { layout: 'fullscreen' },
} satisfies Meta<typeof Loader>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Default: Story = { args: { src: 'http://localhost:{port}' } };
```

### 10. Create src/models/schemas.ts
```typescript
import * as z from 'zod';

export const CursorPaginationSchema = z.object({
    nextCursor: z.string().nullable(),
    previousCursor: z.string().nullable().optional(),
    hasMore: z.boolean(),
    resultCount: z.number(),
    approximateTotal: z.number().nullable().optional(),
});

export const generateCursorPaginatedSchema = <T extends z.ZodTypeAny>(schema: T) =>
    z.object({ data: z.array(schema), pagination: CursorPaginationSchema });

export const ApiProblemDetailsSchema = z.object({
    type: z.string().optional(),
    title: z.string(),
    status: z.number(),
    code: z.string(),
    detail: z.string().optional(),
    traceId: z.string().optional(),
});
```

### 11. Verify
- Confirm the root `package.json` workspaces includes `packages/*` (should be automatic)
- Confirm no port conflicts with existing packages
