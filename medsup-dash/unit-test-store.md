# Unit Test for Transactions Store

## Context
The `useTransactionStore` contains critical business logic, specifically the aggregation of financial data into quarterly summaries and total values. Currently, this logic is untested by the test suite.

Given the complexity of the **Thai Fiscal Year logic** (where Q1 starts in October), it is risky to modify these calculations without automated tests catching regressions.

## Objective
Create a comprehensive unit test file `tests/stores/transactions.spec.ts` to verify that computed getters (`quarterlySummary`, `totalValue`, `averageValue`, etc.) calculate results correctly based on the provided transaction data.

---

## Instructions for the AI Agent

### Step 1: Create the Test File
Create a new file `tests/stores/transactions.spec.ts`.

**Key Requirements:**
1.  Use `vitest` functions (`describe`, `it`, `expect`, `beforeEach`).
2.  Import `createPinia` and `setActivePinia` from `pinia`. This is crucial for creating a clean, isolated store instance for each test.
3.  **No API Mocking Required**: Since we are testing **computed getters** (pure logic), we do not need to mock Supabase or call `fetchByFiscalYear`. We will directly populate the `transactions` state in the test setup.

### Step 2: Implement Test Scenarios

Write the tests following this structure:

```typescript
import { describe, it, expect, beforeEach } from 'vitest';
import { createPinia, setActivePinia } from 'pinia';
import { useTransactionStore } from '@/stores/transactions';

describe('Transaction Store Getters', () => {
  beforeEach(() => {
    // Create a fresh Pinia instance before each test
    setActivePinia(createPinia());
  });

  it('calculates totalValue correctly', () => {
    const store = useTransactionStore();

    // Arrange: Populate state directly
    store.transactions = [
      { id: 1, drug_value: 1000, transaction_date: '2024-01-01', drug_type: 'A', created_at: '', bill_number: null, note: null },
      { id: 2, drug_value: 500, transaction_date: '2024-01-02', drug_type: 'B', created_at: '', bill_number: null, note: null },
    ];

    // Assert
    expect(store.totalValue).toBe(1500);
  });

  it('calculates averageValue correctly', () => {
    const store = useTransactionStore();

    store.transactions = [
      { id: 1, drug_value: 1000, transaction_date: '2024-01-01', drug_type: 'A', created_at: '', bill_number: null, note: null },
      { id: 2, drug_value: 2000, transaction_date: '2024-01-02', drug_type: 'B', created_at: '', bill_number: null, note: null },
    ];

    expect(store.averageValue).toBe(1500); // (1000 + 2000) / 2
  });

  describe('Quarterly Summary (Thai Fiscal Year)', () => {
    it('distributes values into correct quarters based on Thai Fiscal Year', () => {
      const store = useTransactionStore();

      // Arrange: Map dates to Thai Fiscal Quarters
      // Q1: Oct-Dec, Q2: Jan-Mar, Q3: Apr-Jun, Q4: Jul-Sep
      store.transactions = [
        { id: 1, drug_value: 1000, transaction_date: '2023-10-15', drug_type: 'A', created_at: '', bill_number: null, note: null }, // Q1
        { id: 2, drug_value: 2000, transaction_date: '2024-01-15', drug_type: 'B', created_at: '', bill_number: null, note: null }, // Q2
        { id: 3, drug_value: 3000, transaction_date: '2024-05-15', drug_type: 'C', created_at: '', bill_number: null, note: null }, // Q3
        { id: 4, drug_value: 4000, transaction_date: '2024-07-15', drug_type: 'D', created_at: '', bill_number: null, note: null }, // Q4
      ];

      // Assert
      expect(store.quarterlySummary.q1).toBe(1000);
      expect(store.quarterlySummary.q2).toBe(2000);
      expect(store.quarterlySummary.q3).toBe(3000);
      expect(store.quarterlySummary.q4).toBe(4000);
    });

    it('handles empty transaction array gracefully', () => {
      const store = useTransactionStore();
      store.transactions = [];

      expect(store.quarterlySummary.q1).toBe(0);
      expect(store.totalValue).toBe(0);
      expect(store.averageValue).toBe(0);
    });
  });

  it('sorts recentTransactions correctly', () => {
    const store = useTransactionStore();

    store.transactions = [
      { id: 1, drug_value: 0, transaction_date: '2024-01-01', drug_type: 'A', created_at: '', bill_number: null, note: null },
      { id: 2, drug_value: 0, transaction_date: '2024-01-03', drug_type: 'B', created_at: '', bill_number: null, note: null }, // Newest
      { id: 3, drug_value: 0, transaction_date: '2024-01-02', drug_type: 'C', created_at: '', bill_number: null, note: null },
    ];

    const recent = store.recentTransactions;

    expect(recent.length).toBe(3);
    expect(recent[0].id).toBe(2); // Should be the most recent (Jan 3)
  });
});
```

**Important Note on Types:**
The transaction mock objects inside the tests should loosely match the `MedTransaction` structure so TypeScript does not complain about missing properties (you can set optional fields to `null` or empty strings).

---

## Validation

1.  Run `bun run test:unit --run` to execute the new tests.
2.  All tests must pass.
3.  Run `bun run test:coverage` to see if the file is tracked correctly in the coverage report.
4.  Ensure no global variables or side effects leak between tests (the `setActivePinia` call in `beforeEach` handles this).