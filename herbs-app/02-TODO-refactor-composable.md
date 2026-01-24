
# Composable Pattern Refactoring

**Role:** Senior Frontend Architect (Composition API Expert)
**Objective:** Extract the API fetching logic from `HomeView.vue` into a reusable composable (`src/composables/useHerbs.ts`).
**Philosophy:** *Separation of Concerns*. Views should only handle "Presentation". Data fetching and state management should be handled by "Composables".

---

## Phase 1: Directory Structure

1.  **Create Directory:**
    Create a new folder `src/composables` if it doesn't exist.

---

## Phase 2: Create the Composable

1.  **Create File:**
    *   **Path:** `src/composables/useHerbs.ts`

2.  **Implementation Details:**
    *   **Imports:** `ref` from 'vue', `Herb` from '@/types/Herb', `herbsService` from '@/services/herbs-service'.
    *   **State Definition:**
        *   `herbs`: Array to store the fetched data.
        *   `loading`: Boolean to track the request status.
        *   `error`: String to store error messages (null if no error).
    *   **Action:**
        *   `fetchHerbs`: Async function that calls the service.
    *   **Return:** Return an object containing reactive state and actions (destructuring friendly).

3.  **Code Template:**
    ```typescript
    // src/composables/useHerbs.ts
    import { ref } from 'vue';
    import type { Herb } from '@/types/Herb';
    import { herbsService } from '@/services/herbs-service';

    /**
     * Composable for managing Herb data fetching logic.
     * Can be used in Views, Components, or other Composables.
     */
    export function useHerbs() {
      // 1. Reactive State
      const herbs = ref<Herb[]>([]);
      const loading = ref<boolean>(false);
      const error = ref<string | null>(null);

      // 2. Fetching Logic
      /**
       * Fetches herbs from the API.
       * Sets loading state and handles errors gracefully.
       */
      const fetchHerbs = async () => {
        loading.value = true;
        error.value = null;

        try {
          const data = await herbsService.getHerbs();
          herbs.value = data;
        } catch (err) {
          console.error('Failed to fetch herbs:', err);
          // Keep error message user-friendly but detailed enough for debug
          error.value = 'Unable to load herb data. Please try again later.';
        } finally {
          loading.value = false;
        }
      };

      // 3. Expose to the world
      return {
        herbs,
        loading,
        error,
        fetchHerbs,
      };
    }
    ```

---

## Phase 3: Refactor `HomeView.vue`

1.  **Cleanup:**
    Remove ALL logic related to data fetching from the `<script setup>` section.
    *   Delete local imports of `herbsService` (if any).
    *   Delete `ref` definitions for `herbs`, `loading`, `error`.
    *   Delete the internal `function fetchHerbs()` logic.
    *   Delete any internal `try/catch` blocks related to API.

2.  **Integrate Composable:**
    *   Import `useHerbs`.
    *   Call it: `const { herbs, loading, error, fetchHerbs } = useHerbs();`.
    *   Call `fetchHerbs()` inside `onMounted` (or leave it if you want lazy loading).

3.  **Refactored Code Example:**
    ```vue
    <!-- src/views/HomeView.vue -->
    <script setup lang="ts">
    import { onMounted } from 'vue';
    import { useHerbs } from '@/composables/useHerbs'; // Import Composable
    import Header from '@/components/Header.vue';
    import HerbCard from '@/components/HerbCard.vue';
    import HerbCardSkeleton from '@/components/HerbCardSkeleton.vue';
    import SearchBar from '@/components/SearchBar.vue';
    import Footer from '@/components/Footer.vue';

    // Use Composable to handle data
    const { herbs, loading, error, fetchHerbs } = useHerbs();

    // Lifecycle hook
    onMounted(() => {
      fetchHerbs();
    });
    </script>

    <template>
      <div class="min-h-screen bg-background">
        <Header />
        <SearchBar
          :categories="[]" 
          @search="(q) => {}" 
          @filter="(c) => {}" 
          @search:="handleSearch" 
        />
        <!-- Note: Search logic remains in View or moves to another composable useFilter() -->

        <!-- Skeleton State -->
        <div v-if="loading" class="container mx-auto px-4 py-6 grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
          <HerbCardSkeleton v-for="n in 6" :key="n" />
        </div>

        <!-- Error State -->
        <div v-else-if="error" class="container mx-auto px-4 py-6 text-center text-red-500">
          <p class="text-xl font-bold">Oops!</p>
          <p>{{ error }}</p>
          <button @click="fetchHerbs" class="mt-4 px-4 py-2 bg-primary text-white rounded">Retry</button>
        </div>

        <!-- Success State -->
        <div v-else class="container mx-auto px-4 py-6 grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
          <HerbCard
            v-for="herb in herbs"
            :key="herb.ID || herb.Name"
            :herb="herb"
          />
        </div>

        <Footer />
      </div>
    </template>
    ```

---

## Phase 4: Safety & Testing (Crucial)

1.  **TypeScript Verification:**
    *   Run `bun run type-check`. Ensure `herbs` is recognized as `Ref<Herb[]>`.

2.  **Unit Testing the Composable:**
    The power of composables is that you can test them **without** mounting a full Vue Component.
    *   **File:** `test/composables/useHerbs.spec.ts`

    ```typescript
    // test/composables/useHerbs.spec.ts
    import { describe, it, expect, vi, beforeEach } from 'vitest';
    import { setActivePinia, createPinia } from 'pinia'; // If using Pinia, otherwise create simple test context
    import { useHerbs } from '@/composables/useHerbs';
    import * as herbsService from '@/services/herbs-service';

    // Mock the service
    vi.mock('@/services/herbs-service');

    describe('useHerbs', () => {
      beforeEach(() => {
        // Reset mocks before each test
        vi.clearAllMocks();
      });

      it('initializes with default state', () => {
        const { herbs, loading, error } = useHerbs();

        expect(herbs.value).toEqual([]);
        expect(loading.value).toBe(false);
        expect(error.value).toBeNull();
      });

      it('fetches herbs successfully', async () => {
        const mockData = [
          { ID: 1, Name: 'Test Herb', Description: 'Test', Usage: 'Test' }
        ];
        vi.spyOn(herbsService, 'herbsService', 'get').mockResolvedValue(mockData); // Mock implementation

        const { fetchHerbs, herbs, loading, error } = useHerbs();

        await fetchHerbs();

        expect(loading.value).toBe(false);
        expect(error.value).toBeNull();
        expect(herbs.value).toEqual(mockData);
      });

      it('handles fetch errors', async () => {
        vi.spyOn(herbsService, 'herbsService', 'get').mockRejectedValue(new Error('Network Error'));

        const { fetchHerbs, error } = useHerbs();

        await fetchHerbs();

        expect(error.value).toBeTruthy();
      });
    });
    ```
    *(Note: Adjust the mock syntax based on your actual implementation of `herbsService`. If it's a class instance vs a singleton object.)*

---

## Phase 5: Further Optimization (Optional but Recommended)

1.  **Deduplication / Caching:**
    Currently, if you call `fetchHerbs()` in two different components, it triggers two API calls.
    *   **Advanced Pattern:** Refactor `useHerbs` to use a **global state** (like a local variable or integrating with `Pinia`) so the data is fetched once and shared.
    *   *Instruction:* For now, stick to the basic implementation above to ensure stability. If `HomeView` is the only consumer, this is safe.

2.  **Filtering Logic:**
    The user might still have "Search" and "Category" logic inside `HomeView`.
    *   **Next Step:** Consider creating another composable `useFilter.ts` to handle `filteredHerbs` logic based on `herbs` array.

## Final Checklist:
- [ ] `src/composables/useHerbs.ts` created.
- [ ] `HomeView.vue` uses the composable instead of local `ref`.
- [ ] `HomeView.vue` is significantly shorter.
- [ ] No TypeScript errors.
- [ ] App runs on `localhost:5173` without breaking features.

**End of Instructions.**