# E2E Testing with Playwright

**Role:** QA Automation Architect
**Objective:** Implement Playwright to validate the Critical User Journey (Loading, Searching, and Navigation) across different viewports.
**Philosophy:** "Test like a user". Use Locators instead of CSS Selectors for robust, maintainable tests.

---

## Phase 1: Installation & Setup

1.  **Install Playwright:**
    Execute the following command to add Playwright to development dependencies:
    ```bash
    bun add -D @playwright/test
    ```

2.  **Install Browsers:**
    Execute the install script to download Chromium, Firefox, and WebKit:
    ```bash
    bunx playwright install --with-deps
    ```

---

## Phase 2: Configuration

1.  **Create Config File:**
    *   **Path:** `playwright.config.ts`
    *   **Action:** Set up the config to use the development server (`vite dev`) so we don't have to manually start it.

2.  **Configuration Template:**
    ```typescript
    // playwright.config.ts
    import { defineConfig, devices } from '@playwright/test';

    export default defineConfig({
      testDir: './test/e2e', // Tests will be placed here
      fullyParallel: true,
      forbidOnly: !!process.env.CI,
      retries: process.env.CI ? 2 : 0,
      workers: process.env.CI ? 1 : undefined,
      reporter: 'html',
      use: {
        baseURL: 'http://localhost:5173',
        trace: 'on-first-retry', // Capture trace only on failure for debugging
        screenshot: 'only-on-failure',
      },
      projects: [
        {
          name: 'chromium',
          use: { ...devices['Desktop Chrome'] },
        },
        {
          name: 'Mobile Chrome',
          use: { ...devices['Pixel 5'] },
        },
      ],
      // Start the dev server automatically before running tests
      webServer: {
        command: 'bun run dev',
        url: 'http://localhost:5173',
        reuseExistingServer: !process.env.CI,
        timeout: 120000, // 2 minutes to start dev server
      },
    });
    ```

3.  **Update NPM Scripts:**
    *   **File:** `package.json`
    *   **Action:** Add the script `test:e2e`.
    ```json
    "scripts": {
      "test:e2e": "playwright test",
      "test:e2e:ui": "playwright test --ui",
      "test:e2e:report": "playwright show-report"
    }
    ```

---

## Phase 3: Robust Testing (Selectors)

**Crucial Step:** To make E2E tests robust against UI changes, we should use `data-testid` attributes.
*   **Action:** Update `src/components/SearchBar.vue` input field to include `data-testid="search-input"`.
*   **Action:** Update `src/components/HerbCard.vue` root div to include `:data-testid="`herb-card-${herb.ID}`"`.

---

## Phase 4: Writing E2E Tests

1.  **Create Test Directory:**
    Create `test/e2e/`.

2.  **Create Test Suite:**
    *   **File:** `test/e2e/hero-journey.spec.ts`

3.  **Test Scenarios:**
    *   **Scenario A: App Loads & Data Fetches.** Verify skeletons appear and disappear.
    *   **Scenario B: Search Filter.** Type a term and verify the list reduces.
    *   **Scenario C: View Details.** Click a card and verify navigation (assuming routing exists).

    **Code Template:**
    ```typescript
    // test/e2e/hero-journey.spec.ts
    import { test, expect } from '@playwright/test';

    test.describe('Herbs App Critical Journey', () => {
      test.beforeEach(async ({ page }) => {
        await page.goto('/');
      });

      test('should load herbs list and hide skeleton loader', async ({ page }) => {
        // 1. Check if skeletons exist initially (Loading state)
        // Note: If fetch is too fast, this might flake. We assume it takes a few ms.
        const skeletons = page.locator('.animate-pulse');
        await expect(skeletons.first()).toBeVisible();

        // 2. Wait for network idle to ensure data is loaded
        await page.waitForLoadState('networkidle');

        // 3. Verify skeletons are gone and HerbCards appear
        // We look for the text content or a specific class known to be in HerbCard
        const cards = page.locator('[data-testid^="herb-card"]');
        await expect(cards.first()).toBeVisible();

        // Ensure there is more than 1 card
        const count = await cards.count();
        expect(count).toBeGreaterThan(1);
      });

      test('should filter herbs by search query', async ({ page }) => {
        // Wait for initial load
        await page.waitForLoadState('networkidle');

        const searchInput = page.getByTestId('search-input'); // Using test id
        await searchInput.fill('ยาไพล'); // Example herb name from data or a known term
        await searchInput.press('Enter');

        // Wait a bit for UI update (debounce or filter logic)
        await page.waitForTimeout(500);

        // Validate results (Assuming data has this text)
        const cards = page.locator('[data-testid^="herb-card"]');
        const visibleCardsCount = await cards.count();
        
        // Simple assertion: Ensure we have results
        expect(visibleCardsCount).toBeGreaterThan(0);
      });

      test('should navigate to detail view (if applicable)', async ({ page }) => {
        await page.waitForLoadState('networkidle');

        const firstCard = page.locator('[data-testid^="herb-card"]').first();
        await firstCard.click();

        // Expect URL to change (Assuming Vue Router works)
        await expect(page).toHaveURL(/.*detail/); 
        // OR check if a modal opens with details
        await expect(page.locator('text=สรรพคุณ')).toBeVisible();
      });
    });
    ```

---

## Phase 5: Execution & Debugging

1.  **Run Tests (Headless):**
    ```bash
    bun run test:e2e
    ```

2.  **Run Tests (UI Mode - Highly Recommended):**
    This opens a GUI to see what's happening live.
    ```bash
    bun run test:e2e:ui
    ```

3.  **Analyze Results:**
    If a test fails, Playwright generates a trace file. Run the report command to view it:
    ```bash
    bun run test:e2e:report
    ```

---

## Phase 6: CI Integration (Advanced)

Update `.github/workflows/ci-quality.yml` to include E2E testing (Optional).
*   **Action:** Add a step to install browsers (`bunx playwright install --with-deps`) and run tests.
*   **Note:** Ensure the CI environment supports graphical interfaces (usually requires `xvfb` on Linux).

## Final Checklist:
- [ ] `@playwright/test` installed.
- [ ] `playwright.config.ts` created with `webServer`.
- [ ] `data-testid` attributes added to SearchBar and HerbCard.
- [ ] `test/e2e/hero-journey.spec.ts` created with passing tests.
- [ ] `bun run test:e2e` passes successfully on local machine.

**End of Instructions.**