# Setup End-to-End Testing with Playwright

## Context
To ensure reliability before every release, we need End-to-End (E2E) tests that simulate real user interactions in a browser. We will use **Playwright** due to its speed, reliability, and modern auto-waiting features.

**Scope of Test:**
1.  User navigates to the app.
2.  User logs in with valid credentials.
3.  User lands on the Dashboard.
4.  User interacts with the Fiscal Year dropdown.
5.  User verifies that the Dashboard data reflects the change.

---

## Instructions for the AI Agent

### Step 1: Install Playwright
Run the installation command in the terminal.
This command will install Playwright and add the necessary configuration files (`playwright.config.ts`, `tests/e2e` folder) automatically.

```bash
bunx -y @playwright/test init
```

*   When prompted, confirm defaults (TypeScript, End-to-End, 'tests' folder).

### Step 2: Configure Playwright to Run Vite Dev Server
Open the newly created `playwright.config.ts`.

**Objective:**
Configure Playwright to launch the Vite dev server automatically before running tests, so we don't have to start `bun run dev` manually.

**Update the configuration file:**

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:5173',
    trace: 'on-first-retry',
  },

  // Projects configuration
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
  ],

  // Start the local dev server before running the tests
  webServer: {
    command: 'bun run dev',
    url: 'http://localhost:5173',
    reuseExistingServer: !process.env.CI,
    timeout: 120 * 1000,
  },
});
```

### Step 3: Create the Dashboard E2E Test
Create a new file `tests/e2e/dashboard.spec.ts`.

**Requirements:**
-   Use environment variables (`process.env.TEST_EMAIL`, `process.env.TEST_PASSWORD`) for credentials to avoid hardcoding secrets in the repository.
-   Use Playwright's Locators (`getByPlaceholder`, `getByRole`) for robust selection.
-   Ensure the test checks for specific Thai text present in the UI (e.g., "Dashboard รายงานมูลค่ายาสนับสนุน").

**Code to Add:**

```typescript
import { expect, test } from '@playwright/test';

test.describe('Authentication & Dashboard Flow', () => {
  test.beforeEach(async ({ page }) => {
    // Base URL is defined in playwright.config.ts, so we go to root
    await page.goto('/');
  });

  test('User can log in and interact with Fiscal Year selector', async ({ page }) => {
    // 1. Verify Redirect to Login
    await expect(page).toHaveURL(/.*login/);
    await expect(page.getByRole('heading', { name: 'Welcome Back' })).toBeVisible();

    // 2. Perform Login
    // Use environment variables for credentials
    await page.getByPlaceholder('pharmacist@sabot.hospital').fill(process.env.TEST_EMAIL || 'admin@test.com');
    await page.getByPlaceholder('••••••••').fill(process.env.TEST_PASSWORD || 'password');
    
    // Click Sign In
    await page.getByRole('button', { name: 'Sign In' }).click();

    // 3. Verify Dashboard Load
    // Wait for network idle or specific dashboard element to ensure data is loaded
    await expect(page.getByText('Dashboard รายงานมูลค่ายาสนับสนุน')).toBeVisible();
    
    // Verify KPI Cards are present
    await expect(page.getByText('มูลค่ารวมทั้งปี')).toBeVisible();
    await expect(page.getByText('จำนวนรายการทั้งหมด')).toBeVisible();

    // 4. Interact with Fiscal Year Selector
    // Get the current value of the dropdown (e.g., "ปี 2568")
    const dropdown = page.locator('select');
    const currentYearText = await dropdown.inputValue();

    // Change selection to a different index (e.g., second option)
    // Note: The exact options depend on the years generated in the app
    await dropdown.selectOption({ index: 1 }); 

    // 5. Verify Data Reload
    // We verify that the "Loading" state appeared and disappeared, or simply that the URL/context updated.
    // A simple check is ensuring we are still on the dashboard and data elements remain.
    await expect(page.getByText('สรุปรายไตรมาส (Quarterly Report)')).toBeVisible();
  });
});
```

### Step 4: Update Package.json Scripts
Open `package.json` and add the E2E scripts to the `"scripts"` section.

**Add:**

```json
"test:e2e": "playwright test",
"test:e2e:ui": "playwright test --ui",
"test:e2e:report": "playwright show-report"
```

---

## Validation

1.  **Run the Test in UI Mode (Recommended for debugging):**
    ```bash
    bun run test:e2e:ui
    ```
    *   The browser should open, and you can watch the test execute step-by-step.

2.  **Run Headless (CI Mode):**
    ```bash
    bun run test:e2e
    ```

3.  **Verification:**
    *   If you do not provide `.env` variables for `TEST_EMAIL`/`TEST_PASSWORD`, the test will fail or use the hardcoded fallback.
    *   Ensure you have a real user in your Supabase database that matches the credentials used in the test.

4.  **Safety Note:** Ensure `webServer` in the config does not conflict with an already running `bun run dev` on port 5173. If you run tests manually, stop your dev server first to let Playwright manage it.