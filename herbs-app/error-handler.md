# Global Error Handling & User Feedback System

**Role:** Frontend Reliability Engineer
**Objective:** Implement a robust error feedback loop using `vue-toastification` for user notifications and an `ErrorBoundary` component to prevent app-wide crashes.
**Philosophy:** "Fail Gracefully". Users should never see a blank screen. They should see a toast or a fallback component explaining what happened.

---

## Phase 1: Installation

1.  **Install Dependencies:**
    Execute the command:
    ```bash
    bun add vue-toastification
    bun add -d @types/vue-toastification
    ```

---

## Phase 2: Global Toast Configuration

1.  **Configure Plugin:**
    *   **File:** `src/main.ts`
    *   **Action:** Import `Toast` and CSS. Register it as a plugin.
    *   **Styling:** Configure the Toast to match the App's Teal (`#2c7a7b`) theme.

    **Code Integration:**
    ```typescript
    // src/main.ts
    import { createApp } from 'vue';
    import { createPinia } from 'pinia'; // Assuming Pinia is installed
    import Toast, { POSITION } from 'vue-toastification';
    import 'vue-toastification/dist/index.css'; // REQUIRED: Import styles

    import App from './App.vue';
    import router from './router';
    import './assets/main.css';

    // Toast Configuration
    const toastOptions = {
      position: POSITION.TOP_CENTER,
      timeout: 5000,
      closeOnClick: true,
      pauseOnFocusLoss: true,
      pauseOnHover: true,
      draggable: true,
      draggablePercent: 0.6,
      showCloseButtonOnHover: false,
      hideProgressBar: false,
      closeButton: 'button',
      icon: true,
      rtl: false,
      // Custom styles to match Tailwind theme
      toastClassName: 'vue-toastification-custom',
      bodyClassName: ['vue-toastification-custom-body'],
    };

    const app = createApp(App);
    const pinia = createPinia();

    app.use(pinia);
    app.use(router);
    app.use(Toast, toastOptions); // Add this

    app.mount('#app');
    ```

---

## Phase 3: Create Error Boundary Component

Vue 3 uses `onErrorCaptured` lifecycle to handle errors within a component tree. We will create a reusable wrapper.

1.  **Create Component:**
    *   **File:** `src/components/ErrorBoundary.vue`

2.  **Implementation:**
    *   Use `onErrorCaptured` to catch errors from child components.
    *   Use `useToast` to notify the user.
    *   Render a fallback UI.

    **Code Template:**
    ```vue
    <!-- src/components/ErrorBoundary.vue -->
    <script setup lang="ts">
    import { onErrorCaptured } from 'vue';
    import { useToast } from 'vue-toastification';
    import { ref } from 'vue';

    const error = ref<Error | null>(null);
    const toast = useToast();

    // Capture errors from children components
    onErrorCaptured((err, instance, info) => {
      console.error('Caught by ErrorBoundary:', err);
      error.value = err;

      // Show a toast to the user
      toast.error('Something went wrong in this section. Please refresh.', {
        timeout: false, // Keep toast visible
        closeOnClick: false,
      });

      // IMPORTANT: Return false to stop the error from propagating to the global error handler
      return false;
    });

    // Retry mechanism (Optional but good UX)
    const resetError = () => {
      error.value = null;
    };
    </script>

    <template>
      <div class="min-h-full w-full">
        <!-- Fallback UI if error exists -->
        <div v-if="error" class="flex flex-col items-center justify-center p-8 bg-red-50 text-red-600 rounded-lg border border-red-200 h-64">
          <h2 class="text-2xl font-bold mb-2">Oops! Something broke.</h2>
          <p class="mb-4 opacity-80">We encountered an unexpected error in this section.</p>
          <button
            @click="resetError"
            class="px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700 transition-colors shadow-md"
          >
            Try Again
          </button>
        </div>

        <!-- Normal Content Slot if no error -->
        <slot v-else />
      </div>
    </template>
    ```

---

## Phase 4: Application-Wide Error Handler

1.  **Modify `App.vue`:**
    We need to wrap the main application logic so that any unhandled promise rejections or global errors trigger a toast.

    **Code Update:**
    ```vue
    <!-- src/App.vue -->
    <script setup lang="ts">
    import { onErrorCaptured } from 'vue';
    import { useToast } from 'vue-toastification';
    import ErrorBoundary from '@/components/ErrorBoundary.vue';
    import Footer from '@/components/Footer.vue';

    const toast = useToast();

    // Global error handler for unhandled errors falling through components
    onErrorCaptured((err) => {
      console.error('Global Error:', err);
      toast.error('Critical Error: Please refresh the page.');
      // Return false to stop propagation
      return false;
    });

    // Also catch global window errors (optional)
    window.addEventListener('error', (event) => {
      toast.error('Application Error detected.');
      console.error(event);
    });
    </script>

    <template>
      <div class="flex flex-col min-h-screen font-sans bg-background text-text-main">
        <!-- Wrap the main content with ErrorBoundary -->
        <ErrorBoundary>
          <router-view />
        </ErrorBoundary>
        <Footer />
      </div>
    </template>
    ```

---

## Phase 5: Integration with Composables (Data Fetching Errors)

Now we need to use the Toast system when the API fails (e.g., in `useHerbs.ts`).

1.  **Update `src/composables/useHerbs.ts`:**
    *   Import `useToast`.
    *   In the `catch` block, trigger the toast.

    **Refactored Logic:**
    ```typescript
    // src/composables/useHerbs.ts
    import { ref } from 'vue';
    import { useToast } from 'vue-toastification'; // Import
    import type { Herb } from '@/types/Herb';
    import { herbsService } from '@/services/herbs-service';

    export function useHerbs() {
      const herbs = ref<Herb[]>([]);
      const loading = ref<boolean>(false);
      const error = ref<string | null>(null);

      const toast = useToast(); // Initialize toast

      const fetchHerbs = async () => {
        loading.value = true;
        error.value = null;

        try {
          const data = await herbsService.getHerbs();
          herbs.value = data;
        } catch (err) {
          console.error('Failed to fetch herbs:', err);
          error.value = 'Unable to load herb data. Please try again later.';

          // ADD THIS: Show toast notification
          toast.error('Failed to load herbs. Check your internet connection.', {
            timeout: 5000,
          });
        } finally {
          loading.value = false;
        }
      };

      return {
        herbs,
        loading,
        error,
        fetchHerbs,
      };
    }
    ```

---

## Phase 6: Safety & Styling Check

1.  **CSS Tweaks (Optional):**
    The default toast might not perfectly match your `teal-700` theme. You can override specific classes in `src/assets/main.css` if needed, but the default is usually clean enough.

2.  **Testing Error Handling:**
    *   **Network Error:** Turn off your internet and open the app. You should see a Red Toast notification.
    *   **Component Error:** Temporarily introduce a syntax error (e.g., accessing undefined variable) inside `HerbCard.vue`. Verify that the `ErrorBoundary` catches it and shows the "Oops! Something broke" UI instead of crashing the whole page.

3.  **Type Check:**
    Run `bun run type-check` to ensure imports are correct.

## Final Checklist:
- [ ] `vue-toastification` installed and configured in `main.ts`.
- [ ] CSS imported in `main.ts`.
- [ ] `src/components/ErrorBoundary.vue` created.
- [ ] `App.vue` wrapped with ErrorBoundary.
- [ ] `src/composables/useHerbs.ts` updated to use `toast.error`.
- [ ] App runs smoothly and handles errors gracefully.

**End of Instructions.**