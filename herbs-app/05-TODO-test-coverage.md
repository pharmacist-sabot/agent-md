# Service Layer & Utility Testing Strategy

**Role:** Senior QA Engineer / Testing Architect
**Objective:** Achieve high code coverage for the business logic layer (`services` and `utils`) before moving to UI testing. Focus on isolation, mocking external dependencies, and handling edge cases.
**Philosophy:** "Test behavior, not implementation details". Tests should be fast, reliable, and independent of network calls.

---

## Phase 1: Environment & Mocking Setup

1.  **Verify Testing Framework:**
    Ensure `vitest`, `happy-dom`, and `@vue/test-utils` are installed (from your `package.json`). Yes, they are.

2.  **Mocking Strategy:**
    We **MUST** mock network requests. **Do not** allow tests to hit the real Google Sheets API.
    *   Use `vi.stubGlobal('fetch', ...)` to mock the global `fetch` object used by `herbs-service.ts`.

---

## Phase 2: Unit Testing the Service Layer

**Target File:** `src/services/herbs-service.ts`

1.  **Create Test File:**
    *   **Path:** `test/services/herbs-service.spec.ts`

2.  **Instruction for AI:**
    *   Read `src/services/herbs-service.ts` to understand its implementation (likely uses `fetch`).
    *   Write tests for the `getHerbs()` function.

3.  **Test Scenarios to Cover:**
    *   **Success:** Should return an array of `Herb` objects when the API returns valid JSON.
    *   **Success (Empty):** Should return an empty array when API returns `[]`.
    *   **Network Error:** Should throw an error or return empty data when `fetch` fails (404, 500).
    *   **Data Integrity:** Ensure the returned structure matches the TypeScript interface (if applicable).

4.  **Code Template:**
    ```typescript
    // test/services/herbs-service.spec.ts
    import { describe, it, expect, vi, beforeEach } from 'vitest';
    import { herbsService } from '@/services/herbs-service';

    describe('Herbs Service', () => {
      beforeEach(() => {
        vi.clearAllMocks();
      });

      it('fetches herbs data successfully', async () => {
        // Mock data
        const mockResponse = [
          { ID: 1, Name: 'Mock Herb', Description: 'Test', Usage: 'Test' }
        ];

        // Mock global fetch
        vi.stubGlobal('fetch', vi.fn(() =>
          Promise.resolve({
            ok: true,
            json: () => Promise.resolve(mockResponse),
          } as Response)
        ));

        const result = await herbsService.getHerbs();

        expect(result).toEqual(mockResponse);
        expect(fetch).toHaveBeenCalledTimes(1);
      });

      it('handles network errors gracefully', async () => {
        // Mock fetch failure
        vi.stubGlobal('fetch', vi.fn(() => Promise.reject('Network Error')));

        // Assuming your service catches the error or throws it.
        // Adjust this expectation based on your actual error handling strategy.
        await expect(herbsService.getHerbs()).rejects.toThrow();
      });

      it('handles empty response', async () => {
        vi.stubGlobal('fetch', vi.fn(() =>
          Promise.resolve({
            ok: true,
            json: () => Promise.resolve([]),
          } as Response)
        ));

        const result = await herbsService.getHerbs();
        expect(result).toHaveLength(0);
      });
    });
    ```

---

## Phase 3: Refactoring for Testability (Utilities)

**Observation:** In `src/components/HerbCard.vue`, there is a local function `truncateText`.
**Problem:** Logic inside a Component is hard to unit test because it requires mounting the entire Vue component.
**Solution:** Extract `truncateText` into a utility file.

1.  **Create Utility File:**
    *   **Path:** `src/utils/textUtils.ts` (Create folder `src/utils` if it doesn't exist).

2.  **Instruction for AI:**
    *   **Extract Logic:** Move the `truncateText` function from `HerbCard.vue` to `src/utils/textUtils.ts`.
    *   **Export:** Named export: `export function truncateText(text: string | undefined, maxLength: number): string`.
    *   **Update Component:** Update `HerbCard.vue` to import `truncateText` from the utils file.

3.  **Write Unit Tests for Utils:**
    *   **Path:** `test/utils/textUtils.spec.ts`

    **Test Scenarios:**
    *   **Standard Truncation:** Returns string with `...` if length > maxLength.
    *   **Short Text:** Returns original string if length <= maxLength.
    *   **Undefined/Null:** Returns empty string safely.
    *   **Exact Length:** Returns original string if length == maxLength.

    **Code Template:**
    ```typescript
    // test/utils/textUtils.spec.ts
    import { describe, it, expect } from 'vitest';
    import { truncateText } from '@/utils/textUtils';

    describe('Text Utilities', () => {
      it('truncates text longer than max length', () => {
        const result = truncateText('This is a very long description', 10);
        expect(result).toBe('This is a...');
      });

      it('returns original text if shorter than max length', () => {
        const result = truncateText('Short', 10);
        expect(result).toBe('Short');
      });

      it('handles undefined input safely', () => {
        const result = truncateText(undefined, 10);
        expect(result).toBe('');
      });

      it('handles exact length limit', () => {
        const text = '12345';
        const result = truncateText(text, 5);
        expect(result).toBe('12345');
      });
    });
    ```

---

## Phase 4: Configuration & Execution

1.  **Vitest Config Check:**
    Ensure `vitest.config.ts` (or inside `vite.config.ts`) is set to find test files correctly.
    ```typescript
    // vite.config.ts (Test section)
    test: {
      globals: true,
      environment: 'happy-dom',
      include: ['test/**/*.spec.ts'], // Ensure this matches our new paths
    }
    ```

2.  **Run Tests:**
    Execute the tests to ensure nothing is broken:
    ```bash
    bun run test:unit
    ```

3.  **Check Coverage (Optional but Recommended):**
    If coverage is enabled in `vite.config.ts`, run:
    ```bash
    bun run test:coverage
    ```
    *   Verify that `src/services/herbs-service.ts` and `src/utils/textUtils.ts` show high coverage (close to 100%).

---

## Phase 5: Safety Checklist (Before Submitting)

*   [ ] Logic moved from `HerbCard.vue` to `src/utils/textUtils.ts`.
*   [ ] `HerbCard.vue` updated to use the new import.
*   [ ] `test/services/herbs-service.spec.ts` created and tests `fetch` mocking.
*   [ ] `test/utils/textUtils.spec.ts` created and covers edge cases.
*   [ ] `bun run test:unit` passes all tests.
*   [ ] App still builds successfully (`bun run build`).

**End of Instructions.**