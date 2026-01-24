# Refactor Auth Logout Flow

## Context
The current authentication logout flow uses `window.location.reload()` inside the Pinia store. This is a brute-force method that disrupts the Single Page Application (SPA) experience, causes a white screen flash, and is considered an anti-pattern in modern Vue applications.

## Objective
Refactor the `auth.logout` function and its usage in `AppNavbar` to remove the hard reload. Instead, utilize Vue Router's programmatic navigation (`router.replace`) and ensure application state is reset cleanly without browser refreshes.

## Safety & Standards
1.  **No Hard Reloads**: Do not use `window.location.reload()` or `window.location.href`.
2.  **Router History Hygiene**: Use `router.replace('/login')` instead of `router.push`. This prevents the user from navigating back to the authenticated dashboard using the browser's "Back" button after logging out.
3.  **State Sanitization**: Ensure that sensitive user data in other stores (e.g., `transactions`) is cleared upon logout to prevent data leakage if the browser cache is inspected.

---

## Instructions for the AI Agent

### Step 1: Refactor `src/stores/auth.ts`

Modify the `logout` function.
1.  Remove the line `window.location.reload()`.
2.  Keep the `user.value = null` assignment.
3.  **(Crucial)** Import the `useTransactionStore` and reset its state (clear transactions) to ensure no medical data remains in memory after logout.

**Expected Code Change:**

```typescript
// src/stores/auth.ts
import { defineStore } from 'pinia';
import { ref } from 'vue';
import { supabase } from '@/services/supabase';
import { useTransactionStore } from './transactions'; // Import other stores to reset them

export const useAuthStore = defineStore('auth', () => {
  const user = ref<User | null>(null);
  const loading = ref(false);

  // ... (init and login functions remain unchanged)

  async function logout() {
    try {
      // 1. Sign out from Supabase
      await supabase.auth.signOut();
      
      // 2. Reset Auth State
      user.value = null;

      // 3. Reset Application State (Security Best Practice)
      // Clear sensitive data from other stores to prevent data leaks
      const transactionStore = useTransactionStore();
      // Note: Since 'transactions' is a Setup Store, we manually clear the ref/array
      // If the store has a specific reset function, use it. Otherwise, modify state directly.
      transactionStore.transactions = []; 
      
    } catch (error) {
      console.error('Logout Error:', error);
      // We still want to redirect the user even if Supabase API fails locally
      user.value = null;
    }
    // 4. NO window.location.reload() here.
    // Navigation is handled by the component calling this function.
  }

  return { user, loading, init, login, logout };
});
```

### Step 2: Update `src/components/common/AppNavbar.vue`

Modify the `handleLogout` function.
1.  Change `router.push('/login')` to `router.replace('/login')`.
2.  Ensure `await` is used correctly.

**Expected Code Change:**

```vue
<!-- src/components/common/AppNavbar.vue -->
<script setup lang="ts">
// ... (imports remain unchanged)

const authStore = useAuthStore();
const router = useRouter();

async function handleLogout() {
  await authStore.logout();
  
  // Use 'replace' to clear history stack.
  // This prevents the user from hitting 'Back' and seeing the dashboard again.
  router.replace('/login');
}
</script>

<!-- ... (template remains unchanged) -->
```

### Step 3: Verify Router Guards (Optional Check)

Review `src/router/index.ts`. Ensure the logic handles the transition smoothly.
- The `beforeEach` guard checks `authStore.user`.
- Since we set `user.value = null` in Step 1, the guard will correctly see the user as unauthenticated.
- No changes should be needed here, but verify it logs/logic doesn't conflict.

## Validation
After applying these changes, the system should:
1.  Clicking Logout immediately clears the screen and shows the Login page.
2.  The browser URL changes to `/login` instantly without a white flash.
3.  Clicking the browser's "Back" button on the Login page **does not** return to the Dashboard.
4.  The Transactions list is empty if the user logs back in (starts fresh).