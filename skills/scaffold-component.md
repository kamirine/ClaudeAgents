---
description: "Scaffold a React component with Storybook stories and MSW mock handler stubs. Creates .tsx component file, .stories.tsx with required states (Default, Loading, Empty, Error), and MSW handler entry."
args: "package=<package-name> name=<ComponentName>"
---

# Scaffold React Component with Storybook

Create a React component with Storybook stories and MSW mock stubs for a dashboard MFE.

## Arguments

- `package` — MFE package name (e.g., `dashboard-home`, `transaction-explorer`)
- `name` — Component name in PascalCase (e.g., `KpiCards`, `TransactionGrid`, `AlertsFeed`)

## Steps

### 1. Create component file

File: `C:\GIT\Micros\payments-mfe\packages\{package}\src\components\{name}.tsx`

```typescript
import { FC } from 'react';
import { Flex, Text } from '@servicetitan/anvil2';

interface {name}Props {
    // TODO: Define props
}

export const {name}: FC<{name}Props> = (props) => {
    return (
        <Flex direction="column" gap={4}>
            <Text variant="heading-md">{name}</Text>
            {/* TODO: Implement component */}
        </Flex>
    );
};

{name}.displayName = '{name}';
```

### 2. Create Storybook stories

File: `C:\GIT\Micros\payments-mfe\packages\{package}\src\components\{name}.stories.tsx`

```typescript
import type { Meta, StoryObj } from '@storybook/react';
import { http, HttpResponse } from 'msw';
import { {name} } from './{name}';

const meta = {
    title: 'MFEs/{PackagePascal}/{name}',
    component: {name},
    parameters: {
        layout: 'padded',
    },
    tags: ['autodocs'],
} satisfies Meta<typeof {name}>;

export default meta;
type Story = StoryObj<typeof meta>;

// Required state: Default (with data)
export const Default: Story = {
    args: {
        // TODO: Default props with realistic data
    },
};

// Required state: Loading
export const LoadingState: Story = {
    parameters: {
        msw: {
            handlers: [
                // Delayed response to show loading state
                http.get('/publicapi/v2/...', async () => {
                    await new Promise(r => setTimeout(r, 3000));
                    return HttpResponse.json({ data: [], pagination: { nextCursor: null, hasMore: false, resultCount: 0 } });
                }),
            ],
        },
    },
};

// Required state: Empty (zero results)
export const Empty: Story = {
    args: {
        // TODO: Props that result in empty state
    },
};

// Required state: Error
export const ErrorState: Story = {
    parameters: {
        msw: {
            handlers: [
                http.get('/publicapi/v2/...', () =>
                    HttpResponse.json(
                        { code: 'RATE_LIMIT_EXCEEDED', title: 'Too Many Requests', status: 429 },
                        { status: 429 }
                    )),
            ],
        },
    },
};

// Accessibility story
export const KeyboardNavigation: Story = {
    args: { /* same as Default */ },
    play: async ({ canvasElement }) => {
        // TODO: Test keyboard navigation
    },
};
```

### 3. Update MSW handlers

Add relevant handler to `packages/{package}/src/mocks/handlers.ts`:

```typescript
// Add to handlers array:
http.get('/publicapi/v2/{endpoint}', () =>
    HttpResponse.json({
        data: [/* mock data */],
        pagination: { nextCursor: null, hasMore: false, resultCount: 0 }
    })),
```

### 4. Requirements checklist

- [ ] Component uses functional component with hooks (no class component)
- [ ] ARIA labels on all interactive elements (`aria-label`, `role`)
- [ ] Keyboard navigation support (tabIndex, onKeyDown)
- [ ] Uses Anvil2 components where possible
- [ ] Stories cover: Default, Loading, Empty, Error states
- [ ] MSW mocks return `CursorPaginatedResponse` shape (not offset pagination)
- [ ] MSW error mocks return `ApiProblemDetails` shape
- [ ] Component handles loading, error, and empty states gracefully
- [ ] No Kendo CSS imports (imported once in host app)
