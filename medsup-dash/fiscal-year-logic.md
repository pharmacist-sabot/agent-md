# Refactor Fiscal Year Logic into Reusable Utility

## Context
The logic for calculating the **Thai Fiscal Year** (starting in October) is currently duplicated in two places:
1.  `src/stores/transactions.ts` (default initialization)
2.  `src/views/dashboard/OverviewView.vue` (dropdown generation)

The logic `new Date().getFullYear() + (new Date().getMonth() >= 9 ? 1 : 0)` should be extracted into a centralized, type-safe utility function. This adheres to the **Don't Repeat Yourself (DRY)** principle and makes the codebase easier to maintain.

## Logic Definition
The Thai Fiscal Year starts on **October 1st**.
- **Months Jan - Sep (0-8):** Fiscal Year = Current Calendar Year.
- **Months Oct - Dec (9-11):** Fiscal Year = Current Calendar Year + 1.

---

## Instructions for the AI Agent

### Step 1: Create the Utility Function
Open `src/utils/formatters.ts` and add the following function at the end of the file.

**Requirements:**
- Accepts `Date` or `string` inputs.
- Handles edge cases (invalid dates) gracefully by returning the current year.
- Includes JSDoc for world-class documentation.

**Code to Add:**

```typescript
/**
 * Calculates the Thai Fiscal Year based on the given date.
 * Thai Fiscal Year starts on October 1st (Month index 9).
 *
 * @param date - The input date (Date object or ISO string). Defaults to current time if not provided.
 * @returns The calculated Fiscal Year (number).
 *
 * @example
 * getFiscalYear(new Date('2024-10-01')) // Returns 2025
 * getFiscalYear(new Date('2024-09-30')) // Returns 2024
 */
export function getFiscalYear(date: Date | string = new Date()): number {
  const d = new Date(date);

  // Handle invalid date inputs (e.g., 'Invalid Date')
  if (isNaN(d.getTime())) {
    console.warn('[getFiscalYear] Invalid date input provided. Returning current year.');
    return new Date().getFullYear();
  }

  const currentYear = d.getFullYear();
  const monthIndex = d.getMonth(); // 0 = Jan, 9 = Oct

  // If month is October (9) or later, add 1 to the year
  return monthIndex >= 9 ? currentYear + 1 : currentYear;
}
```

### Step 2: Refactor `src/stores/transactions.ts`
Locate the `selectedFiscalYear` ref initialization inside the `defineStore` setup block.

**Action:**
Replace the inline calculation with a call to the new utility.

**Change from:**
```typescript
const selectedFiscalYear = ref<number>(new Date().getFullYear() + (new Date().getMonth() >= 9 ? 1 : 0));
```

**Change to:**
```typescript
import { getFiscalYear } from '@/utils/formatters';

// ... inside defineStore
const selectedFiscalYear = ref<number>(getFiscalYear());
```

### Step 3: Refactor `src/views/dashboard/OverviewView.vue`
Locate the `currentFiscalYear` variable definition inside the `<script setup>`.

**Action:**
1.  Import `getFiscalYear` from `@/utils/formatters`.
2.  Replace the inline logic.

**Change from:**
```typescript
const currentFiscalYear = new Date().getFullYear() + (new Date().getMonth() >= 9 ? 1 : 0);
```

**Change to:**
```typescript
import { getFiscalYear } from '@/utils/formatters';

const currentFiscalYear = getFiscalYear();
```

### Step 4: Update Unit Tests (Best Practice)
Open `tests/utils/formatters.spec.ts`.
Add a test suite for the new function to ensure behavior is correct across different dates.

**Code to Add:**

```typescript
import { describe, expect, it } from 'vitest';
import { getFiscalYear } from '@/utils/formatters';

// ... existing tests ...

describe('getFiscalYear', () => {
  it('returns current year for January', () => {
    expect(getFiscalYear('2024-01-15')).toBe(2024);
  });

  it('returns current year for September', () => {
    expect(getFiscalYear('2024-09-30')).toBe(2024);
  });

  it('returns next year for October', () => {
    expect(getFiscalYear('2024-10-01')).toBe(2025);
  });

  it('returns next year for December', () => {
    expect(getFiscalYear('2024-12-31')).toBe(2025);
  });

  it('handles Date objects correctly', () => {
    expect(getFiscalYear(new Date('2024-10-01'))).toBe(2025);
  });

  it('handles invalid dates gracefully', () => {
    // Expecting a valid number (current year) even with bad input, not NaN
    expect(getFiscalYear('invalid-date')).toBeDefined();
    expect(typeof getFiscalYear('invalid-date')).toBe('number');
  });
});
```

---

## Validation
After executing these steps:
1.  No logic changes should occur in the application UI. The selected year in the dropdown should remain the same.
2.  Run `bun run test:unit` to ensure all new tests pass.
3.  Run `bun run type-check` to ensure TypeScript compilation succeeds.
4.  The application should build successfully without warnings about unused code or missing imports.