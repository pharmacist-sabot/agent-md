# Pinia State Management Integration

**Role:** Senior Vue.js Architect / Refactoring Specialist
**Objective:** Integrate Pinia to manage global application state (Herbs, Search, Filter, Loading) to decouple logic from Views and improve maintainability.
**Strict Rules:**
1.  **Zero Breaking Changes:** The application must compile without errors after each step.
2.  **Type Safety:** Use TypeScript strictly. No `any` types allowed.
3.  **Separation of Concerns:** Do not duplicate logic. Use existing `src/services/herbs-service.ts`.
4.  **World-Class Standards:** Use Vue 3 Composition API, `<script setup>`, and modular store patterns.

---

## Phase 1: Environment Setup & Installation

1.  **Install Pinia:**
    Execute the following command:
    ```bash
    bun add pinia
    ```

2.  **Initialize Pinia in App:**
    *   **File:** `src/main.ts`
    *   **Action:** Import `createPinia` and use it before mounting the app.
    *   **Code Change:**
        ```typescript
        import { createPinia } from 'pinia'; // Add this
        import App from './App.vue';
        import router from './router';
        import './assets/main.css';

        const app = createApp(App);
        const pinia = createPinia(); // Create instance

        app.use(pinia); // Add this line
        app.use(router);

        app.mount('#app');
        ```

---

## Phase 2: Create the Core Store

1.  **Create Directory Structure:**
    Create a new folder `src/stores`.

2.  **Create Herb Store:**
    *   **File:** `src/stores/herbStore.ts`
    *   **Type Reference:** Import `Herb` type from `@/types/Herb`.
    *   **Logic Design:**
        *   **State:** `herbs` (array), `filteredHerbs` (computed), `searchQuery` (ref), `selectedCategory` (ref), `loading` (ref), `error` (ref).
        *   **Getters:** `filteredHerbs` should automatically filter the list based on `searchQuery` and `selectedCategory`. **Do not** filter manually in the View.
        *   **Actions:**
            *   `fetchHerbs()`: Use `herbsService.getHerbs()`. Handle loading/error states here.
            *   `setSearchQuery(query: string)`
            *   `setCategory(category: string)`

    *   **Template Code (Use this guide to implement):**
        ```typescript
        // src/stores/herbStore.ts
        import { ref, computed } from 'vue';
        import { defineStore } from 'pinia';
        import type { Herb } from '@/types/Herb';
        import { herbsService } from '@/services/herbs-service';

        export const useHerbStore = defineStore('herb', () => {
          // State
          const herbs = ref<Herb[]>([]);
          const loading = ref(false);
          const error = ref<string | null>(null);
          const searchQuery = ref('');
          const selectedCategory = ref('');

          // Getters (Computed)
          const filteredHerbs = computed(() => {
            let result = herbs.value;

            // Filter by Category
            if (selectedCategory.value) {
              result = result.filter(h => h.Category === selectedCategory.value);
            }

            // Filter by Search Query (Name or Description)
            if (searchQuery.value) {
              const lowerQuery = searchQuery.value.toLowerCase();
              result = result.filter(h =>
                h.Name.toLowerCase().includes(lowerQuery) ||
                (h.Description && h.Description.toLowerCase().includes(lowerQuery))
              );
            }

            return result;
          });

          // Actions
          async function fetchHerbs() {
            loading.value = true;
            error.value = null;
            try {
              herbs.value = await herbsService.getHerbs();
            } catch (err) {
              console.error(err);
              error.value = 'Failed to fetch herbs data';
            } finally {
              loading.value = false;
            }
          }

          function setSearchQuery(query: string) {
            searchQuery.value = query;
          }

          function setCategory(category: string) {
            selectedCategory.value = category;
          }

          return {
            herbs,
            loading,
            error,
            searchQuery,
            selectedCategory,
            filteredHerbs,
            fetchHerbs,
            setSearchQuery,
            setCategory
          };
        });
        ```

---

## Phase 3: Refactor Views

### 1. Refactor `HomeView.vue`
*   **Goal:** Remove all local state (`ref`, `computed`, logic) related to herbs and search. Delegate everything to `useHerbStore`.
*   **Steps:**
    1.  Import `useHerbStore`.
    2.  Initialize store: `const herbStore = useHerbStore()`.
    3.  Remove: `const filteredHerbs = ref([])`, `const searchQuery = ref('')`, `function onSearch()`, `function onFilter()`, and `watch` effects if any.
    4.  Remove: Logic that manually calculates `filteredHerbs`.
    5.  Keep: `import SearchBar from '@/components/SearchBar.vue';`
    6.  Lifecycle: Call `herbStore.fetchHerbs()` inside `onMounted`.
    7.  Template: Update props passing. Since we are removing emit logic, see Phase 4.

**Before (Example Logic to Remove):**
```vue
<script setup lang="ts">
// ... existing imports
const herbStore = useHerbStore();
// Remove local state
// const filteredHerbs = ref<Herb[]>([]);
// const searchQuery = ref('');

// Remove local functions
// function onSearch() { ... }
</script>
```

**After (Clean Code):**
```vue
<script setup lang="ts">
import { onMounted } from 'vue';
import { useHerbStore } from '@/stores/herbStore';
import HerbCard from '@/components/HerbCard.vue';
import HerbCardSkeleton from '@/components/HerbCardSkeleton.vue';
import SearchBar from '@/components/SearchBar.vue';

const herbStore = useHerbStore();

onMounted(() => {
  herbStore.fetchHerbs();
});
</script>

<template>
  <div class="min-h-screen bg-background">
    <Header />
    <SearchBar />
    <!-- Use store directly -->
    <div v-if="herbStore.loading" class="...">
      <HerbCardSkeleton v-for="i in 6" :key="i" />
    </div>
    <div v-else-if="herbStore.error" class="...">
      Error: {{ herbStore.error }}
    </div>
    <div v-else class="...">
      <HerbCard
        v-for="herb in herbStore.filteredHerbs"
        :key="herb.ID || herb.Name"
        :herb="herb"
      />
    </div>
    <Footer />
  </div>
</template>
```

---

## Phase 4: Refactor Components (SearchBar.vue)

*   **Goal:** Make `SearchBar` interact with the Pinia store directly. This removes the need for `defineEmits` and props drilling.
*   **File:** `src/components/SearchBar.vue`
*   **Steps:**
    1.  Import `useHerbStore`.
    2.  Initialize store.
    3.  Remove: `defineEmits<{ (e: 'search', query: string): void }>();`.
    4.  Remove: `defineProps<{ categories: string[] }>();` (Optional: You can get categories from the store if you compute them there, or keep props if they are purely UI-driven. For simplicity and robustness, let's compute categories in the store or just keep them dynamic from the list). *Recommendation: Compute categories from `herbStore.herbs`.*
    5.  Update `onSearch`: Call `herbStore.setSearchQuery(searchQuery.value)`.
    6.  Update `onFilter`: Call `herbStore.setCategory(selectedCategory.value)`.

**Refactored Script Logic:**
```vue
<script setup lang="ts">
import { ref, computed } from 'vue';
import { useHerbStore } from '@/stores/herbStore';

const herbStore = useHerbStore();

// Local UI State (for input fields)
const searchQuery = ref('');
const selectedCategory = ref('');

// Computed Categories (Dynamic from store data)
const categories = computed(() => {
  const cats = new Set(herbStore.herbs.map(h => h.Category));
  return Array.from(cats).sort();
});

function onSearch() {
  herbStore.setSearchQuery(searchQuery.value);
}

function onFilter() {
  herbStore.setCategory(selectedCategory.value);
}
</script>
```

---

## Phase 5: Safety Checks & Testing

1.  **Type Check:**
    Run `bun run type-check`. Ensure there are **zero** TypeScript errors. Fix type mismatches immediately.

2.  **Unit Test Updates (Crucial):**
    *   The file `test/views/HomeView.spec.ts` will fail because the logic moved.
    *   **Instruction:** Update the test file. Mock `useHerbStore` and test that `filteredHerbs` are rendered correctly based on store state.
    *   **Snippet:**
        ```typescript
        import { setActivePinia, createPinia } from 'pinia';
        // ... other imports

        describe('HomeView.vue', () => {
          beforeEach(() => {
            setActivePinia(createPinia());
          });

          it('renders herb cards from store', () => {
            const wrapper = mount(HomeView);
            // Expectations based on mocked store...
          });
        });
        ```

3.  **Manual Testing:**
    *   Run `bun run dev`.
    *   Verify: The app loads, skeletons appear, data fetches.
    *   Verify: Search filters work instantly.
    *   Verify: Category dropdown works.

## Final Checklist before considering "DONE":
- [ ] `src/main.ts` uses Pinia.
- [ ] `src/stores/herbStore.ts` exists and exports `useHerbStore`.
- [ ] `HomeView.vue` is simplified to just calling `fetchHerbs` and rendering.
- [ ] `SearchBar.vue` writes to the store directly (no emits).
- [ ] No build errors (`bun run build`).
- [ ] TypeScript check passes (`bun run type-check`).

**End of Instructions.**