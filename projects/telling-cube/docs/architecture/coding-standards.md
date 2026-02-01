# Coding Standards

**Document Version:** 1.0
**Extracted From:** architecture.md
**Purpose:** Code style, linting, and formatting rules for tellingCube

---

## TypeScript Configuration

**File:** `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "dom", "dom.iterable"],
    "jsx": "preserve",
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "allowJs": true,
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "forceConsistentCasingInFileNames": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "incremental": true,
    "plugins": [{ "name": "next" }],
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

**Key TypeScript Rules:**
- ✅ **Strict mode enabled** (`strict: true`)
- ✅ **No unchecked indexed access** - Arrays and objects require explicit undefined checks
- ✅ **Explicit return types** for exported functions (recommended but not enforced)
- ⚠️ **Avoid `any` types** where possible (ESLint warns on usage)

## ESLint Configuration

**File:** `.eslintrc.json`

```json
{
  "extends": [
    "next/core-web-vitals",
    "plugin:@typescript-eslint/recommended"
  ],
  "rules": {
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/no-explicit-any": "warn",
    "@typescript-eslint/explicit-function-return-type": "off",
    "react/prop-types": "off",
    "react/react-in-jsx-scope": "off"
  }
}
```

**Run linting:**
```bash
pnpm lint
```

## Prettier Configuration

**File:** `.prettierrc`

```json
{
  "semi": false,
  "singleQuote": true,
  "trailingComma": "es5",
  "tabWidth": 2,
  "printWidth": 100,
  "arrowParens": "avoid"
}
```

**Format code:**
```bash
pnpm format
```

## Naming Conventions

### Files

- **Components**: `PascalCase.tsx` (e.g., `FinanceView.tsx`)
- **Utilities**: `kebab-case.ts` (e.g., `format-currency.ts`)
- **Types**: `kebab-case.ts` (e.g., `api.ts`)
- **API routes**: `route.ts` (Next.js convention)

### Variables

- **Variables/functions**: `camelCase`
- **Components/types**: `PascalCase`
- **Constants**: `SCREAMING_SNAKE_CASE`

### Example

```typescript
// Constants
const MAX_EVENTS_PER_SCENARIO = 400
const GENERATION_TIMEOUT_MS = 90_000

// Functions
function calculateNetMargin(revenue: number, costs: number): number {
  return (revenue - costs) / revenue
}

// Components
export function FinanceView({ data }: FinanceViewProps) {
  return <div>...</div>
}

// Types
interface FinanceViewProps {
  data: FinanceViewResponse
}
```

## Code Organization

### Import Order

```typescript
// 1. React/Next.js
import { useState } from 'react'
import { useRouter } from 'next/navigation'

// 2. External libraries
import { z } from 'zod'
import { prisma } from '@prisma/client'

// 3. Internal utilities
import { formatCurrency } from '@/lib/utils/format-currency'
import { GenerationService } from '@/lib/services/generation.service'

// 4. Components
import { FinanceView } from '@/components/views/FinanceView'

// 5. Types
import type { FinanceViewResponse } from '@/types/api'

// 6. Styles (if any)
import styles from './styles.module.css'
```

## Comments

### When to Comment

- ✅ **Complex business logic**
- ✅ **IBCS-specific requirements**
- ✅ **Event-sourcing validation rules**
- ✅ **Workarounds for known issues**

### When NOT to Comment

- ❌ **Obvious code** (`// Increment counter` for `count++`)
- ❌ **Type definitions** (types are self-documenting)

### Example

```typescript
// ✅ GOOD: Explains WHY
// Allow 1% tolerance for revenue reconciliation due to Decimal rounding in Prisma aggregations
const RECONCILIATION_TOLERANCE = 0.01

// ❌ BAD: Explains WHAT (obvious from code)
// Create a new scenario
const scenario = await prisma.scenario.create({ ... })
```

## Git Commit Messages

### Format

```
<type>: <subject>

<body>
```

### Types

- `feat:` New feature
- `fix:` Bug fix
- `refactor:` Code refactoring
- `docs:` Documentation
- `test:` Tests
- `chore:` Build/tooling

### Example

```
feat: Add revenue reconciliation validation

Implements ValidationService.checkRevenueReconciliation() which ensures
Sales transaction amounts match cash movement inbound totals within 1% tolerance.

Closes #12
```

## Function Return Types

**Best Practice:** Always specify return types for exported functions.

```typescript
// ✅ GOOD: Explicit return type
export function calculateRevenue(events: Event[]): number {
  return events.reduce((sum, e) => sum + (e.amountEur ?? 0), 0)
}

// ⚠️ ACCEPTABLE: TypeScript can infer, but explicit is better for exports
export function formatDate(date: Date) {
  return date.toISOString().split('T')[0]
}
```

## Error Handling

**Best Practice:** Use try-catch blocks for all async operations.

```typescript
export async function generateScenario(type: string): Promise<ScenarioResponse> {
  try {
    const events = await claudeService.generate(type)
    const scenarioId = await prisma.scenario.create({ data: { type } })
    return { scenarioId, events }
  } catch (error) {
    console.error('Generation failed:', error)
    throw new Error('Failed to generate scenario')
  }
}
```
