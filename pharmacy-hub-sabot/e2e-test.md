# AGENTS.md

> **Target:** World-class Full-Stack Developer / QA Engineer  
> **Objective:** Implement Robust E2E Testing using Playwright for "Pharmacy Hub Sabot"  
> **Constraint:** Do NOT modify application logic in a way that breaks existing functionality.  
> **Standard:** Production-grade, reliable, and maintainable.

---

## 1. Context & Tech Stack Analysis

- **Framework:** Vue 3.5 (Composition API) with `<script setup>`.
- **Routing:** Vue Router 4.
- **State:** Pinia.
- **Styling:** Tailwind CSS 4.
- **Hosting:** Firebase Hosting (SPA).
- **Current Testing:** Vitest (Unit Tests) with 80% coverage threshold.
- **Critical External Dependency:** `public/data/resources.json` (Loaded via `fetch`).

---

## 2. Tool Selection: Playwright vs Cypress

**Decision: Use Playwright.**
- **Why:** Native TypeScript support, superior auto-waiting mechanisms (less flaky tests), multi-browser support (Chrome, Firefox, Safari), and better integration with Vite.
- **Architecture:** We will use the `webServer` option in `playwright.config.ts` to spin up the Vite dev server automatically during tests.

---

## 3. Implementation Steps

### Step 3.1: Installation & Setup

1.  **Install Dependencies:**
    Run the following command in the project root:
    ```bash
    npm install -D @playwright/test
    ```
    *(Do NOT install `@playwright/vue` unless specific component mounting is needed; we prefer full browser E2E).*

2.  **Initialize Playwright:**
    Generate the config file:
    ```bash
    npx playwright init
    ```

### Step 3.2: Configuration (`playwright.config.ts`)

Replace the generated config with this tailored configuration for our repository.

**Requirements:**
- Use `webServer` to run `npx vite` on port 5173.
- Set `baseURL` to `http://localhost:5173`.
- Configure viewport sizes for Mobile (iPhone 13) and Desktop.
- **Crucial:** Use `trace: 'retain-on-failure'` for debugging CI failures.

### Step 3.3: Test Architecture (Page Object Model)

Do not write selectors directly in the test files. Create a "Pages" directory structure:

```
tests/
  └── e2e/
      ├── pages/
      │   ├── AppPage.ts           (Common elements like Header, Sidebar)
      │   └── HomePage.ts          (Resource Grid, Search)
      └── specs/
          ├── navigation.spec.ts   (Sidebar navigation)
          ├── search.spec.ts       (Search functionality)
          └── tools.spec.ts        (Tool card interactions)
```

**Best Practice:** Use `page.getByRole()`, `page.getByTestId()`, or `page.getByText()`. AVOID CSS Selectors (`#id`, `.class`) unless absolutely necessary.

### Step 3.4: Test Data Strategy

- **Do NOT touch production data.**
- The app loads `public/data/resources.json` statically.
- **Approach:** Since the app fetches real JSON, ensure the dev server serves this file correctly.
- If you need to test specific states (e.g., empty search), manipulate the UI (typing in the search box) rather than mocking the API endpoint, unless mocking significantly improves stability.

### Step 3.5: Writing the Specs (Core User Flows)

You must implement tests for these flows:

1.  **Navigation Flow:**
    - Click Sidebar items (Tools, Reports, External).
    - **Assert:** The URL updates correctly (`/tools`, `/reports`).
    - **Assert:** The active state of the sidebar button changes visually.

2.  **Search Flow:**
    - Type a keyword in the search bar (e.g., "Warfarin").
    - **Assert:** The grid updates to show only matching cards.
    - Clear the search.
    - **Assert:** All cards reappear.

3.  **Tool Interaction Flow:**
    - Locate the "Warfarin Calculator" card.
    - Click the "Open" (เปิดใช้งาน) button/link.
    - **Safety:** Since this is an external link, `page.waitForURL` might wait forever if the external site is down.
    - **Solution:** Use `page.route` to intercept the navigation or simply verify `href` attributes if navigation is too risky.
    - **Better Solution:** Verify the `href` attribute matches the expected URL in `resources.json`.

### Step 3.6: Adding Test IDs (If needed)

If the existing DOM lacks accessible roles for Playwright, add `data-testid` attributes to the components:
- `AppSidebar.vue`: `data-testid="nav-all"`, `data-testid="nav-tool"`, etc.
- `AppHeader.vue`: `data-testid="search-input"`, `data-testid="mobile-search-btn"`.
- `ResourceCard.vue`: `data-testid="tool-card"` (e.g., `data-testid="tool-card-med-safety"`).

---

## 4. Safety & Integrity Protocols

1.  **Non-Destructive:**
    - E2E tests MUST NOT mutate `public/data/resources.json`.
    - E2E tests MUST NOT modify `stores/ui.ts` state permanently (use `unmount` or cleanup).

2.  **Idempotency:**
    - Ensure running tests 5 times in a row yields the same result.

3.  **Mocking External Services:**
    - Tools like "MedSafety Net" link to Google Scripts.
    - **Action:** Mock the external URLs if you intend to click them, or just verify attributes.
    - **Warning:** Do not let the browser actually navigate to `script.google.com` during tests; it will hang due to auth requirements.

4.  **Performance:**
    - Run tests in "Headless" mode by default.

---

## 5. CI/CD Integration

Update `.github/workflows/ci.yml` to add a new job or step:

```yaml
e2e:
  name: E2E Tests
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with:
        node-version: 18
    - name: Install dependencies
      run: npm ci
    - name: Install Playwright
      run: npx playwright install --with-deps
    - name: Run Playwright tests
      run: npx playwright test
    - name: Upload Playwright Report
      if: always()
      uses: actions/upload-artifact@v4
      with:
        name: playwright-report
        path: playwright-report/
```

---

## 6. Expected Output

After the task is complete, the repository must have:
1.  A working `playwright.config.ts`.
2.  A `tests/e2e/` directory with at least 3 specs.
3.  A passing local run: `npx playwright test`.
4.  Documentation on how to open the HTML report (`npx playwright show-report`).
5.  No build errors in the main application.