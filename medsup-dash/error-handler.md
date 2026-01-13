# Implement Global Error Handler & Logging Service

## Context
Currently, error handling is sporadic and relies on `console.error` within individual components or stores. In a production environment, we cannot rely on users' browsers' consoles to detect bugs. We need a centralized error handler that captures runtime errors globally and forwards them to a monitoring service (like Sentry) while providing helpful debugging information in development.

## Objective
Implement `app.config.errorHandler` in Vue to create a global Error Boundary.
1.  Create a dedicated Logger Utility to handle error formatting and service dispatch (mocking Sentry integration).
2.  Register this handler in `src/main.ts`.
3.  Ensure the handler logs meaningful details: Error instance, Vue component instance, and hook info (e.g., "render function").

---

## Instructions for the AI Agent

### Step 1: Create the Logger Utility
Create a new file `src/utils/logger.ts`.

**Requirements:**
-   Export a function `logError` that accepts Vue's error handler arguments.
-   **Development Mode**: Output detailed, colorful logs to the console.
-   **Production Mode**: Send error data to an external service (e.g., Sentry). Since we are not installing Sentry yet, create a clear placeholder comment where the integration code would live.
-   Check for `import.meta.env.PROD` to toggle behavior.

**Code to Add:**

```typescript
// src/utils/logger.ts

/**
 * Global Error Logger
 * Captures errors and routes them to the console (Dev) or an external service (Prod).
 */

export function logError(err: unknown, instance: any, info: string) {
  // 1. Extract Error Details
  const errorName = err instanceof Error ? err.name : 'Unknown Error';
  const errorMessage = err instanceof Error ? err.message : JSON.stringify(err);
  const componentName = instance?.$options?.name || instance?.$options?.__name || 'Anonymous Component';

  // 2. Development Mode: Detailed Console Logging
  if (!import.meta.env.PROD) {
    console.group(`%c[Vue Error]`, 'color: #ef4444; font-weight: bold;');
    console.error(`Error: ${errorName}`);
    console.error(`Message: ${errorMessage}`);
    console.log(`Component: ${componentName}`);
    console.log(`Info: ${info}`);
    console.groupEnd();
    return;
  }

  // 3. Production Mode: Send to Monitoring Service (e.g., Sentry)
  // TODO: Integrate Sentry.io or similar service here.
  // Sentry.captureException(err, { tags: { component: componentName } });
  
  console.warn('An error occurred and has been logged to the monitoring service.');
}
```

### Step 2: Register in Main Application
Open `src/main.ts`.

**Requirements:**
-   Import the `logError` function.
-   Attach it to `app.config.errorHandler`.

**Change to:**

```typescript
// src/main.ts
import { createPinia } from 'pinia';
import { createApp } from 'vue';

import App from './App.vue';
import router from './router';
import './assets/main.css';

// Import the global error handler
import { logError } from './utils/logger';

const app = createApp(App);

app.use(createPinia());
app.use(router);

// Register Global Error Handler
// This catches errors from component renderers, watchers, and lifecycle hooks
app.config.errorHandler = (err, instance, info) => {
  logError(err, instance, info);
};

// Optional: Handle unhandled promise rejections globally
// window.addEventListener('unhandledrejection', (event) => {
//   logError(event.reason, null, 'Unhandled Promise Rejection');
// });

app.mount('#app');
```

### Step 3: (Optional) Cleanup Existing Stores
While not strictly required to remove every `console.error` immediately, it is good practice to rely on the global handler for uncaught exceptions.
You may review `src/stores/auth.ts` and `src/stores/transactions.ts`.
- Keep specific `console.log` for intentional debugging flow traces.
- Ensure actual errors (caught in `catch` blocks) are re-thrown or passed to a logger if they are critical.

---

## Validation

1.  Run `bun run dev`.
2.  Intentionally cause an error in a component (e.g., add a `throw new Error('Test Crash')` inside `OverviewView.vue`).
3.  Open the Browser Console.
4.  **Expected Result:**
    -   You should see a grouped console log labeled `[Vue Error]`.
    -   It should show the Error Name, Message, Component Name (e.g., `OverviewView`), and the Info string.
    -   The app might freeze the specific component, but the logger has successfully captured the event.
5.  Ensure the app still mounts and other parts of the UI remain responsive (unless the error is in the root App.vue).