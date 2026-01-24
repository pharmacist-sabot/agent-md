# AGENTS.md: API Service Layer Refactoring

**Objective:**  
Extract direct Supabase API calls from the Pinia Store (`src/stores/`) into a dedicated Service Layer (`src/services/`).
**Goal:**  
Implement the Separation of Concerns principle. The Store should manage **State**, while the Service should manage **Data Fetching & Transformation**.
**Standard:**  
Clean Architecture, Modular Design, Testability, TypeScript Best Practices.

---

## 1. Phase I: Directory Structure

Create the directory to house all API services.

**Action:**
Create folder `src/services/`.

---

## 2. Phase II: Creating the Drug Service

This layer will encapsulate all Supabase logic. The Store will not know about SQL, table names, or API structures anymore.

**New File:** `src/services/drugService.ts`

**Instructions:**
1.  Create `src/services/drugService.ts`.
2.  Import the Supabase client and Drug types.
3.  Export a `searchDrugs` function that handles the query logic.
4.  **Standard Practice:** Ensure the function explicitly returns typed data and throws errors correctly for the UI to handle.

**Code Content:**
```typescript
import { supabase } from '@/lib/supabaseClient'
import type { Drug } from '@/types/drug'

/**
 * Drug Service
 * Handles all direct communication with the Supabase backend regarding drugs.
 */

/**
 * Searches for drugs by generic or trade name.
 * @param query - The search string.
 * @returns Promise<Drug[]> - An array of matching drugs.
 * @throws Error if the API call fails.
 */
export const searchDrugs = async (query: string): Promise<Drug[]> => {
  if (!query || query.trim() === '') {
    return []
  }

  try {
    // Direct database call
    const { data, error } = await supabase
      .from('drugs')
      .select('*')
      .or(`generic_name.ilike.%${query}%,trade_name.ilike.%${query}%`)
      .limit(50)

    if (error) {
      // Log server-side or detailed error, then throw for the UI/Store
      console.error('Service Error fetching drugs:', error)
      throw new Error(error.message)
    }

    return (data as Drug[]) || []
  } catch (err) {
    // Re-throw to allow the Pinia Store to handle the error state
    throw err
  }
}

/**
 * Fetches a single drug by ID.
 * (Future-proofing for a detail view that fetches by ID instead of search result)
 */
export const getDrugById = async (id: string): Promise<Drug | null> => {
  try {
    const { data, error } = await supabase
      .from('drugs')
      .select('*')
      .eq('id', id)
      .single()

    if (error) {
      console.error('Service Error fetching drug by ID:', error)
      throw new Error(error.message)
    }

    return data as Drug
  } catch (err) {
    throw err
  }
}
```

---

## 3. Phase III: Refactoring the Pinia Store

Now we update the Store to use the new Service. The Store becomes much lighter and focuses on state management (loading, error, results).

**File:** `src/stores/drugStore.ts`

**Instructions:**
1.  Remove the direct import of `supabase` from this file.
2.  Import `searchDrugs` from `@/services/drugService`.
3.  Update the `fetchDrugs` action to call the service function.

**Code Refactoring:**
```typescript
import { ref, computed } from 'vue'
import { defineStore } from 'pinia'
import { searchDrugs } from '@/services/drugService' // 👈 Import Service
import type { Drug } from '@/types/drug'

export const useDrugStore = defineStore('drug', () => {
  // State remains the same
  const drugs = ref<Drug[]>([])
  const searchQuery = ref<string>('')
  const selectedDrug = ref<Drug | null>(null)
  const isLoading = ref<boolean>(false)
  const error = ref<string | null>(null)

  // Getters remain the same
  const hasResults = computed(() => drugs.value.length > 0)
  const isEmpty = computed(() => !isLoading.value && !hasResults.value && searchQuery.value.length > 0)

  // Actions
  const fetchDrugs = async (query: string) => {
    if (!query || query.trim() === '') {
      drugs.value = []
      return
    }

    searchQuery.value = query
    isLoading.value = true
    error.value = null

    try {
      // 👈 CALL SERVICE INSTEAD OF DIRECT SUPABASE CALL
      const results = await searchDrugs(query)
      drugs.value = results
    } catch (err: any) {
      // The service threw the error; we catch it here to update UI state
      console.error('Store Error:', err)
      error.value = err.message || 'Failed to fetch drugs'
      drugs.value = []
    } finally {
      isLoading.value = false
    }
  }

  const selectDrug = (drug: Drug) => {
    selectedDrug.value = drug
  }

  const clearSelection = () => {
    selectedDrug.value = null
  }

  const resetSearch = () => {
    drugs.value = []
    searchQuery.value = ''
    error.value = null
    isLoading.value = false
  }

  return {
    drugs,
    searchQuery,
    selectedDrug,
    isLoading,
    error,
    hasResults,
    isEmpty,
    fetchDrugs,
    selectDrug,
    clearSelection,
    resetSearch
  }
})
```

---

## 4. Phase IV: Verification & System Integrity

**Action 1: Type Check**
Ensure the imports are resolving correctly with the `@` alias.
```bash
npm run type-check
```

**Action 2: Unit Tests Update (Critical)**

Because we refactored the logic, the existing mock in `drugStore.test.ts` targeting the `supabase` client **will now fail** because the Store no longer imports Supabase.

**Instructions for Agent:**
Update the `src/stores/__tests__/drugStore.test.ts` file.

**New Mocking Strategy:**
Instead of mocking `../../lib/supabaseClient`, mock the service itself.

**Updated Test Snippet:**
```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { setActivePinia, createPinia } from 'pinia'
import { useDrugStore } from '../drugStore'
import type { Drug } from '../../types/drug'

// 👇 MOCK THE SERVICE INSTEAD OF SUPABASE
vi.mock('@/services/drugService', () => ({
  searchDrugs: vi.fn()
}))

// Import the mocked function to control its behavior in tests
import { searchDrugs } from '@/services/drugService'

describe('useDrugStore (with Service Layer)', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
    vi.clearAllMocks()
  })

  it('should fetch drugs via the service', async () => {
    const store = useDrugStore()
    const mockDrugs: Drug[] = [
      {
        id: '1',
        generic_name: 'Paracetamol',
        trade_name: 'Tylenol',
        created_at: ''
      }
    ]

    // Mock the service to return our data
    vi.mocked(searchDrugs).mockResolvedValue(mockDrugs)

    await store.fetchDrugs('Para')

    // Assertions
    expect(searchDrugs).toHaveBeenCalledWith('Para') // Verify correct args passed to service
    expect(store.drugs).toEqual(mockDrugs)
    expect(store.isLoading).toBe(false)
  })

  it('should handle service errors', async () => {
    const store = useDrugStore()
    
    // Mock service rejection
    vi.mocked(searchDrugs).mockRejectedValue(new Error('API Down'))

    await store.fetchDrugs('test')

    expect(store.error).toBe('API Down')
  })
})
```

---

## 5. Benefits of this Architecture

1.  **Single Responsibility:** The Service handles *how* to get data. The Store handles *what* to do with the data (loading state, error state).
2.  **Reusability:** If you need to fetch drugs in a different component (e.g., a "Recently Viewed" feature) without using the Store, you can simply import `searchDrugs` from the Service.
3.  **Testability:** Unit tests for the Store are now much simpler. You just mock a function (`searchDrugs`) instead of the complex Supabase client chain.

---

## 6. Summary of Actions

1.  Create folder `src/services/`.
2.  Create `src/services/drugService.ts` containing `searchDrugs` and `getDrugById` functions.
3.  Refactor `src/stores/drugStore.ts` to import and use the service functions.
4.  Update `src/stores/__tests__/drugStore.test.ts` to mock the `drugService` instead of `supabaseClient`.
5.  Run `npm run type-check` and `npm run test` to ensure zero regressions.