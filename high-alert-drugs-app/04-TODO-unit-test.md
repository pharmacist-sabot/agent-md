# Vitest Setup & Business Logic Unit Testing

**Objective:**  
Install Vitest (Vite-native testing framework) and write comprehensive Unit Tests for the drug search business logic.
**Goal:**  
Achieve **>= 80% Code Coverage** on critical search logic and state management without touching the production database.
**Standard:**  
World-Class Unit Testing (AAA Pattern, Mocking, Isolation, Code Coverage).

---

## 1. Phase I: Dependency Installation

Install Vitest, Vue Testing Utils, Coverage tools, and a DOM environment.

**Action:**
```bash
npm install -D vitest @vue/test-utils @vitest/coverage-v8 happy-dom
```

*   `vitest`: The test runner.
*   `@vue/test-utils`: Helper functions for Vue components.
*   `@vitest/coverage-v8`: Code coverage reporter (Fast and accurate).
*   `happy-dom`: A lightweight DOM implementation (Faster than jsdom).

---

## 2. Phase II: Test Configuration

Update Vite config to enable Vitest and configure coverage thresholds.

**File:** `vite.config.ts`

**Instructions:**
1.  Locate the existing `defineConfig` export.
2.  Add a `test` property configuration.
3.  Set up the coverage provider and exclusion rules.

**Code Addition:**
```typescript
import { fileURLToPath, URL } from 'node:url'
import { defineConfig, loadEnv } from 'vite'
import vue from '@vitejs/plugin-vue'
import { VitePWA } from 'vite-plugin-pwa'

export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, process.cwd(), '')

  return {
    plugins: [
      vue(),
      VitePWA({ /* ... existing config ... */ })
    ],
    resolve: {
      alias: {
        '@': fileURLToPath(new URL('./src', import.meta.url))
      }
    },
    
    // 👇 ADD THIS BLOCK FOR VITEST
    test: {
      globals: true, // Allows using 'describe', 'it', 'expect' globally
      environment: 'happy-dom', // Use happy-dom instead of node
      coverage: {
        provider: 'v8', // v8 provider is faster
        reporter: ['text', 'json', 'html'], // Reports in terminal, json, and html file
        exclude: [
          'node_modules/',
          'src/main.ts',
          'src/vite-env.d.ts',
          '*.config.js',
          '*.config.ts',
          'dist/',
        ],
        // World-class standard: Fail if coverage drops below 80%
        statements: 80,
        branches: 80,
        functions: 80,
        lines: 80
      }
    }
  }
})
```

---

## 3. Phase III: Package.json Scripts

Update `package.json` to include test scripts.

**File:** `package.json`

**Instructions:**
Add the following scripts to the `"scripts"` section.

**Code Content:**
```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest run --coverage"
  }
}
```

---

## 4. Phase IV: Business Logic Unit Tests (The Core Task)

We will test the `drugStore` logic directly. This isolates business rules from UI complexity.

**New File:** `src/stores/__tests__/drugStore.test.ts`

**Instructions:**
1.  Create directory `src/stores/__tests__/`.
2.  Create `drugStore.test.ts`.
3.  **Mocking Strategy:** We must mock the Supabase client. We do NOT want to call the real database during tests.
4.  Follow the AAA pattern: **A**rrange (Set up), **A**ct (Run function), **A**ssert (Check result).

**Code Content:**
```typescript
import { setActivePinia, createPinia } from 'pinia'
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest'
import { useDrugStore } from '../drugStore'
import type { Drug } from '../../types/drug'

// 1. MOCK THE SUPABASE CLIENT
// This prevents real API calls and allows us to control the data returned
vi.mock('../../lib/supabaseClient', () => ({
  supabase: {
    from: vi.fn(() => ({
      select: vi.fn(() => ({
        or: vi.fn(() => ({
          limit: vi.fn(() => ({
            data: null,
            error: null
          }))
        }))
      }))
    }))
  }
}))

// Import the mocked supabase to type-check our mocks
import { supabase } from '../../lib/supabaseClient'

describe('useDrugStore', () => {
  beforeEach(() => {
    // Create a fresh Pinia instance for every test to avoid state leakage
    setActivePinia(createPinia())
    
    // Reset mocks before each test
    vi.clearAllMocks()
  })

  it('should have initial state set to defaults', () => {
    const store = useDrugStore()
    
    expect(store.drugs).toEqual([])
    expect(store.searchQuery).toBe('')
    expect(store.selectedDrug).toBe(null)
    expect(store.isLoading).toBe(false)
    expect(store.error).toBe(null)
  })

  it('should NOT fetch drugs if query is empty', async () => {
    const store = useDrugStore()
    
    await store.fetchDrugs('')
    
    expect(store.drugs).toEqual([])
    expect(store.isLoading).toBe(false)
    // Ensure supabase.from was NOT called
    expect(supabase.from).not.toHaveBeenCalled()
  })

  it('should fetch drugs and update state on success', async () => {
    const store = useDrugStore()
    
    // ARRANGE: Mock Data
    const mockDrugs: Drug[] = [
      {
        id: '1',
        generic_name: 'Insulin',
        trade_name: 'Humalog',
        created_at: new Date().toISOString()
      }
    ]

    // Construct the mock chain response
    // @ts-ignore - We are mocking a complex object chain for simplicity
    supabase.from.mockReturnValue({
      select: vi.fn().mockReturnValue({
        or: vi.fn().mockReturnValue({
          limit: vi.fn().mockResolvedValue({
            data: mockDrugs,
            error: null
          })
        })
      })
    })

    // ACT
    await store.fetchDrugs('Insulin')

    // ASSERT
    expect(store.isLoading).toBe(false)
    expect(store.drugs).toEqual(mockDrugs)
    expect(store.searchQuery).toBe('Insulin')
    expect(store.error).toBe(null)
    
    // Verify API was called correctly
    expect(supabase.from).toHaveBeenCalledWith('drugs')
  })

  it('should handle API errors gracefully', async () => {
    const store = useDrugStore()
    
    const mockError = { message: 'Network Error' }

    // Mock failure
    // @ts-ignore
    supabase.from.mockReturnValue({
      select: vi.fn().mockReturnValue({
        or: vi.fn().mockReturnValue({
          limit: vi.fn().mockResolvedValue({
            data: null,
            error: mockError
          })
        })
      })
    })

    await store.fetchDrugs('SomeDrug')

    expect(store.isLoading).toBe(false)
    expect(store.drugs).toEqual([])
    expect(store.error).toBe(mockError.message)
  })

  it('should manage selected drug correctly', () => {
    const store = useDrugStore()
    const mockDrug: Drug = {
      id: '1',
      generic_name: 'Morphine',
      trade_name: 'MS Contin',
      created_at: ''
    }

    store.selectDrug(mockDrug)

    expect(store.selectedDrug).toEqual(mockDrug)

    store.clearSelection()

    expect(store.selectedDrug).toBe(null)
  })

  it('should compute hasResults and isEmpty correctly', () => {
    const store = useDrugStore()
    
    // Default empty state
    expect(store.hasResults).toBe(false)
    expect(store.isEmpty).toBe(false) // Query is empty, so isEmpty is false

    // Simulate a search with no results
    store.drugs = []
    store.searchQuery = 'UnknownDrug'
    store.isLoading = false

    expect(store.hasResults).toBe(false)
    expect(store.isEmpty).toBe(true)

    // Simulate a search with results
    store.drugs = [{ id: '1', generic_name: 'A', trade_name: 'B', created_at: '' }] as Drug[]
    
    expect(store.hasResults).toBe(true)
    expect(store.isEmpty).toBe(false)
  })
})
```

---

## 5. Phase V: Running the Tests & Coverage

After creating the test file, run the verification commands.

**Action 1: Run Tests (Watch Mode)**
```bash
npm run test
```
*Expected:* Vitest should detect the test file and run it successfully. All tests should pass.

**Action 2: Check Coverage Report**
```bash
npm run test:coverage
```
*Expected:*
1.  Terminal output showing a table with percentages.
2.  Ensure `drugStore.ts` coverage is **>= 80%**.
3.  A `coverage` folder should be generated in the root. Open `coverage/index.html` in a browser to see visualized coverage.

---

## 6. Phase VI: Troubleshooting & Best Practices

*   **Mocking Tips:** If your Supabase query chain is complex (`from().select().or().limit()`), you must mock every step in the chain exactly as shown in the "Fetch Success" test. Vitest's `vi.fn()` does not automatically chain.
*   **Async Tests:** Always use `async/await` when testing actions that perform asynchronous operations (API calls).
*   **Testing Components:** While this guide focuses on Logic (Store), you should also test critical UI components (e.g., `DrugSearch.vue`) using `@vue/test-utils` to verify that user inputs (typing) trigger the Store actions correctly.
*   **CI/CD:** Ensure `npm run test:ci` (or just `vitest run`) is added to your GitHub Actions or CI pipeline to prevent broken code from being merged.

---

## 7. Summary of Actions

1.  Install `vitest`, `@vue/test-utils`, `@vitest/coverage-v8`, `happy-dom`.
2.  Update `vite.config.ts` with the `test` block.
3.  Update `package.json` with test scripts.
4.  Create `src/stores/__tests__/drugStore.test.ts` using the provided Mock strategy.
5.  Run `npm run test:coverage`.
6.  Verify Coverage is > 80%.