# Pinia Integration & State Management Refactoring

**Objective:**  
Integrate Pinia as the central state management solution and refactor component-based local state into global stores.
**Goal:**  
Improve maintainability and scalability without breaking existing functionality (Backward Compatible).
**Standard:**  
Vue 3 Best Practices, Composition API, Clean Architecture.

---

## 1. Phase I: Installation & Setup

Execute the following commands to ensure dependencies are present.

**Action:**
```bash
npm install pinia
```

---

## 2. Phase II: Pinia Initialization (Safe Step)

Modify the application entry point to initialize Pinia. This step is non-breaking.

**File:** `src/main.js`

**Instructions:**
1. Import `createPinia` from 'pinia'.
2. Add `.use(createPinia())` to the app instance.

**Expected Code Structure:**
```javascript
import { createApp } from 'vue'
import App from './App.vue'
import { createPinia } from 'pinia' // 👈 1. Import Pinia

import './style.css'

const app = createApp(App)
const pinia = createPinia() // 👈 2. Create Pinia instance

app.use(pinia) // 👈 3. Use Pinia
app.mount('#app')
```

---

## 3. Phase III: Store Definition

Create a centralized store to handle Drug-related logic (Search Results, Selections, Loading States).

**New File:** `src/stores/drugStore.js`

**Instructions:**
1. Create the directory `src/stores/` if it does not exist.
2. Create `drugStore.js`.
3. Use `defineStore` with the Composition API setup function.
4. Import the existing Supabase client.

**Code Content:**
```javascript
import { ref, computed } from 'vue'
import { defineStore } from 'pinia'
import { supabase } from '../lib/supabaseClient'

export const useDrugStore = defineStore('drug', () => {
  // State
  const drugs = ref([])
  const searchQuery = ref('')
  const selectedDrug = ref(null)
  const isLoading = ref(false)
  const error = ref(null)

  // Getters
  const hasResults = computed(() => drugs.value.length > 0)
  const isEmpty = computed(() => !isLoading.value && !hasResults.value && searchQuery.value.length > 0)

  // Actions
  const fetchDrugs = async (query) => {
    if (!query || query.trim() === '') {
      drugs.value = []
      return
    }

    searchQuery.value = query
    isLoading.value = true
    error.value = null

    try {
      // TODO: Ensure the table name matches the actual Supabase table (e.g., 'drugs', 'high_alert_drugs')
      // Using a generic query pattern. Adjust column names (generic_name, trade_name) as necessary.
      const { data, error: fetchError } = await supabase
        .from('drugs') // ⚠️ VERIFY TABLE NAME
        .select('*')
        .or(`generic_name.ilike.%${query}%,trade_name.ilike.%${query}%`)
        .limit(50)

      if (fetchError) throw fetchError

      drugs.value = data || []
    } catch (err) {
      console.error('Error fetching drugs:', err)
      error.value = err.message
      drugs.value = []
    } finally {
      isLoading.value = false
    }
  }

  const selectDrug = (drug) => {
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
    // State
    drugs,
    searchQuery,
    selectedDrug,
    isLoading,
    error,
    // Getters
    hasResults,
    isEmpty,
    // Actions
    fetchDrugs,
    selectDrug,
    clearSelection,
    resetSearch
  }
})
```

---

## 4. Phase IV: Component Refactoring (Strangler Fig Pattern)

Do NOT rewrite all components at once. Refactor `App.vue` first to act as the bridge, then `DrugSearch.vue`.

### A. Refactoring `App.vue`

**File:** `src/App.vue`

**Instructions:**
1. Import `useDrugStore`.
2. Use `storeToRefs` to keep reactivity.
3. Replace local `ref` state with store state.
4. **IMPORTANT:** Keep the Modal UI logic (`isModalOpen`) local to `App.vue` if it is strictly UI-related (e.g., open/close animation) unless specifically requested to move to store. Only move `selectedDrug` data to the store.

**Migration Example (Conceptual):**
```vue
<script setup>
import { ref } from 'vue'
import { useDrugStore } from '@/stores/drugStore'
import { storeToRefs } from 'pinia' // ⚠️ Crucial for keeping reactivity

import DrugSearch from './components/DrugSearch.vue'
import DrugDetailModal from './components/DrugDetailModal.vue'

// Store
const drugStore = useDrugStore()
const { selectedDrug } = storeToRefs(drugStore) // Extract reactive state
const { selectDrug, clearSelection } = drugStore // Extract actions

// Local UI State (Keep local to prevent unnecessary complexity)
const isModalOpen = ref(false)

// Functions
const handleDrugSelect = (drug) => {
  selectDrug(drug) // Store action
  isModalOpen.value = true
}

const handleCloseModal = () => {
  clearSelection() // Store action
  isModalOpen.value = false
}
</script>

<template>
  <header>...</header>

  <main>
    <!-- Pass store state/actions as props if DrugSearch is not yet migrated -->
    <DrugSearch 
      @select="handleDrugSelect"
    />
  </main>

  <!-- Use store state directly in Modal -->
  <DrugDetailModal 
    v-if="isModalOpen && selectedDrug" 
    :drug="selectedDrug" 
    @close="handleCloseModal" 
  />
</template>
```

### B. Refactoring `DrugSearch.vue` (Optional but Recommended)

**File:** `src/components/DrugSearch.vue`

**Instructions:**
If this component currently handles the API call, replace it with the Store call.
1. Import `useDrugStore`.
2. Call `fetchDrugs` from the store on user input.

---

## 5. Phase V: Verification & Safety Checks

After completing the changes:

1.  **Run Dev Server:** `npm run dev`
2.  **Console Check:** Ensure no errors in the browser console.
3.  **Functional Test:**
    *   Type in the search box.
    *   Verify results appear.
    *   Click a drug result.
    *   Verify the modal opens with correct data.
    *   Close the modal.
    *   Verify state clears.

---

## 6. Constraints & Error Handling

*   **Table Name:** The code assumes a table named `drugs`. If the actual table name is different (e.g., `high_alert_medications`), update the `src/stores/drugStore.js` fetch query immediately.
*   **Column Names:** Verify columns `generic_name` and `trade_name` exist. If not, update the `.or()` filter in the store.
*   **Backup:** Before deleting any large chunks of code from the original components, ensure the new Store logic works perfectly. Comment out old code instead of deleting immediately during the first pass.