# Task: Setup End-to-End Testing with Playwright

## Objective
Implement automated End-to-End (E2E) tests to verify the complete user flow (Data Input -> WASM Calculation -> UI Rendering). This ensures the application works as a whole, not just in isolated parts.

**Constraints:**
1.  **No Test-IDs:** Avoid adding `data-testid` attributes to the code if possible. Use Playwright's user-facing locators (roles, text content, labels) to mimic real user behavior.
2.  **Asynchronous Safety:** Explicitly wait for the WASM module to load (check for the "Ready" indicator in the UI) before attempting calculations.

---

## Phase 1: Installation

Install Playwright and necessary dependencies:

```bash
npm install -D @playwright/test
```

Install the browser binaries (required to run tests):

```bash
npx playwright install
```

---

## Phase 2: Configuration (`playwright.config.ts`)

Create a `playwright.config.ts` file in the root directory. This config sets up the local development server and environment.

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:5173',
    trace: 'on-first-retry', // Capture video/trace on failure
    screenshot: 'only-on-failure',
  },

  // Start the Vite Dev Server before running tests
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:5173',
    reuseExistingServer: !process.env.CI,
    timeout: 120 * 1000, // Give time for WASM build
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    // Add 'firefox' and 'webkit' if cross-browser testing is required
  ],
});
```

---

## Phase 3: Test Cases (`e2e/calculation.spec.ts`)

Create a folder `e2e/` and add `calculation.spec.ts`. Implement the following critical scenarios:

### Test 1: Happy Path (Basic Calculation)
Verifies that a standard dose generates a valid result card.

```typescript
import { test, expect } from '@playwright/test';

test.describe('Warfarin Calculation Flow', () => {
  test('should calculate a simple 21mg weekly dose', async ({ page }) => {
    await page.goto('/');

    // 1. Wait for WASM Initialization (Green Dot)
    // The app has a green dot when ready: <span class="bg-green-500 ..."></span>
    await expect(page.locator('.bg-green-500')).toBeVisible({ timeout: 10000 });

    // 2. Input Target Dose
    // Locate the large input field by its placeholder or label
    const doseInput = page.getByPlaceholder('0');
    await doseInput.click();
    await doseInput.fill('21');

    // 3. Click Generate
    await page.getByRole('button', { name: /generate regimen/i }).click();

    // 4. Verify Result Card Appears
    const resultCard = page.locator('.glass-card-white').first();
    await expect(resultCard).toBeVisible();

    // 5. Verify Data Accuracy
    // Expect to see "21.0 mg/wk" in the results
    await expect(page.getByText('21.0 mg/wk')).toBeVisible();
  });
});
```

### Test 2: Error Handling (Zero Dose)
Verifies that invalid inputs trigger the error message.

```typescript
test('should show error for 0 dose', async ({ page }) => {
  await page.goto('/');

  // Wait for WASM
  await expect(page.locator('.bg-green-500')).toBeVisible();

  // Input 0
  await page.getByPlaceholder('0').fill('0');
  await page.getByRole('button', { name: /generate regimen/i }).click();

  // Verify Error Message
  const errorMessage = page.getByText(/กรุณากรอกขนาดยาเป้าหมายให้ถูกต้อง/i);
  await expect(errorMessage).toBeVisible();
});
```

### Test 3: UI Interaction (Pill Selector)
Verifies that toggling pill availability updates the UI state.

```typescript
test('should toggle pill availability', async ({ page }) => {
  await page.goto('/');

  // Find the 1mg pill button
  const pill1mg = page.getByRole('button', { name: /1 mg/i });

  // Check if it's selected (checkmark present) or not based on logic
  // Assuming 1mg is disabled by default based on code: { 1: false }
  await expect(pill1mg).toHaveClass(/grayscale/); // Verify it looks disabled

  // Click to enable
  await pill1mg.click();

  // Verify it is now active (checkmark icon appears)
  await expect(pill1mg.locator('svg')).toBeVisible(); // The checkmark
});
```

---

## Phase 4: Execution & CI Integration

### Local Execution
Add scripts to `package.json`:

```json
{
  "scripts": {
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:e2e:debug": "playwright test --debug"
  }
}
```

*   Run `npm run test:e2e:ui` to see tests running in a browser window with a visual inspector (highly recommended).

### CI Integration
Add the E2E step to your `.github/workflows/ci.yml` (refer to the previous task).

```yaml
# Add this job to your existing CI file
e2e-tests:
  runs-on: ubuntu-latest
  needs: quality-gate # Run only after quality checks pass
  timeout-minutes: 10

  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with:
        node-version: "22"
    
    - name: Install Rust & wasm-pack (Optimized for CI)
      run: |
        curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
        source $HOME/.cargo/env
        cargo install wasm-pack
    
    - name: Install dependencies
      run: npm ci

    - name: Install Playwright Browsers
      run: npx playwright install --with-deps chromium

    - name: Run E2E Tests
      run: npx playwright test

    - name: Upload Playwright Report
      if: always()
      uses: actions/upload-artifact@v4
      with:
        name: playwright-report
        path: playwright-report/
        retention-days: 30
```

## Validation Steps

1.  **Run Locally:** Execute `npm run test:e2e:ui`. You should see the Chromium window open, the app load, and the mouse move automatically to perform actions.
2.  **Trace Viewing:** If a test fails, run `npx playwright show-trace` and open the `.zip` file generated to see a recording of exactly what happened.
3.  **Flakiness Check:** Run the tests 3 times in a row (`npx playwright test --workers=1`) to ensure they pass every time (no timing issues).

## Final Note
Ensure tests wait for UI elements to appear (`await expect(...).toBeVisible()`) rather than using `page.waitForTimeout()`. Playwright's auto-waiting feature is superior and makes tests faster and more stable.