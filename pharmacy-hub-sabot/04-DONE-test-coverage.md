# Quality Assurance & Coverage Boost

> **Role:** QA Automation Engineer / TDD Specialist  
> **Objective:** Achieve >70% Code Coverage by implementing comprehensive Unit Tests for critical business logic and components.  
> **Strict Constraint:** Zero regression. Do not modify production source code (`src/`) logic, only write tests.

---

## 🎯 Target Metrics

- **Coverage Threshold:** 70% - 80%
- **Tools:** Vitest, @vue/test-utils, jsdom
- **Focus Areas:** Pinia Stores, Critical Components, Router Guards.

---

## 🛠 Mission Tasks

### Task 1: Verify Test Environment Configuration

**Action:** Ensure Vitest is configured to generate coverage reports.

1.  Open `vitest.config.ts`.
2.  Verify that `coverage` configuration exists (or add it).
3.  Ensure the `exclude` list does not exclude the critical `src/` folders.

**Configuration Update (Required):**

```typescript
import { configDefaults, defineConfig, mergeConfig } from 'vitest/config';
import viteConfig from './vite.config';
import { fileURLToPath } from 'node:url';

export default mergeConfig(
  viteConfig,
  defineConfig({
    test: {
      globals: true,
      environment: 'jsdom',
      exclude: [...configDefaults.exclude, 'e2e/**'],
      root: fileURLToPath(new URL('./', import.meta.url)),
      coverage: {
        provider: 'v8',
        reporter: ['text', 'json', 'html'],
        include: [
          'src/**/*.{js,ts,vue}',
        ],
        exclude: [
          'src/main.ts',
          'src/env.d.ts',
          'src/**/*.d.ts',
          'src/types/**', // Types are usually inferred
        ],
        all: true, // Check all files, even those without tests
        statements: 70,
        branches: 70,
        functions: 70,
        lines: 70,
      },
    },
  }),
);
```

---

### Task 2: Test Pinia Store (`stores/ui.ts`)

**Impact:** High. Testing state management is cheap and yields high coverage percentage.

1.  Create file: `tests/stores/ui.spec.ts`.
2.  Test initialization, state mutations, and actions.

**Implementation:**

```typescript
import { setActivePinia, createPinia } from 'pinia';
import { describe, it, expect, beforeEach } from 'vitest';

import { useUIStore } from '@/stores/ui';

describe('useUIStore', () => {
  beforeEach(() => {
    setActivePinia(createPinia());
  });

  it('should have initial state', () => {
    const store = useUIStore();
    expect(store.currentTab).toBe('all');
    expect(store.isMobileMenuOpen).toBe(false);
    expect(store.searchQuery).toBe('');
  });

  it('should toggle mobile menu', () => {
    const store = useUIStore();
    expect(store.isMobileMenuOpen).toBe(false);
    
    store.toggleMobileMenu();
    
    expect(store.isMobileMenuOpen).toBe(true);
  });

  it('should update search query', () => {
    const store = useUIStore();
    store.searchQuery = 'warfarin';
    expect(store.searchQuery).toBe('warfarin');
  });

  it('should update current tab', () => {
    const store = useUIStore();
    store.currentTab = 'tool';
    expect(store.currentTab).toBe('tool');
  });
});
```

---

### Task 3: Test Component Logic (`components/common/ResourceCard.vue`)

**Impact:** High. Covers UI rendering logic and prop handling.

1.  Create file: `tests/components/ResourceCard.spec.ts`.
2.  Test rendering based on `isActive` status and props.

**Implementation:**

```typescript
import { describe, it, expect } from 'vitest';
import { mount } from '@vue/test-utils';
import ResourceCard from '@/components/common/ResourceCard.vue';
import type { ResourceItem } from '@/types/resource';

const mockItem: ResourceItem = {
  id: 'test-tool',
  title: 'Test Tool',
  description: 'Test Description',
  iconName: 'Pill',
  url: 'https://example.com',
  isActive: true,
  type: 'tool',
};

describe('ResourceCard', () => {
  it('renders tool title and description', () => {
    const wrapper = mount(ResourceCard, {
      props: { item: mockItem },
    });
    expect(wrapper.text()).toContain('Test Tool');
    expect(wrapper.text()).toContain('Test Description');
  });

  it('shows ONLINE badge when isActive is true', () => {
    const wrapper = mount(ResourceCard, {
      props: { item: { ...mockItem, isActive: true } },
    });
    expect(wrapper.text()).toContain('ONLINE');
  });

  it('shows MAINTENANCE badge and disables link when isActive is false', () => {
    const wrapper = mount(ResourceCard, {
      props: { item: { ...mockItem, isActive: false } },
    });
    expect(wrapper.text()).toContain('MAINTENANCE');
    expect(wrapper.find('a').attributes('href')).toBeUndefined();
  });

  it('renders correct link when isActive is true', () => {
    const wrapper = mount(ResourceCard, {
      props: { item: mockItem },
    });
    const link = wrapper.find('a');
    expect(link.attributes('href')).toBe('https://example.com');
    expect(link.attributes('target')).toBe('_blank');
  });
});
```

---

### Task 4: Test Logic in Component (`components/layout/AppHeader.vue`)

**Impact:** Medium. Tests conditional logic (Greeting based on time).

1.  Create file: `tests/components/AppHeader.spec.ts`.
2.  Mock `Date` to test different greetings.

**Implementation:**

```typescript
import { describe, it, expect, beforeEach, afterEach, vi } from 'vitest';
import { mount } from '@vue/test-utils';
import AppHeader from '@/components/layout/AppHeader.vue';

describe('AppHeader', () => {
  beforeEach(() => {
    vi.useFakeTimers();
  });

  afterEach(() => {
    vi.restoreAllMocks();
  });

  it('shows สวัสดีตอนเช้า before 12:00', () => {
    const date = new Date(2025, 0, 1, 9, 0, 0);
    vi.setSystemTime(date);
    
    const wrapper = mount(AppHeader);
    // Assuming the greeting is in the h1 element
    expect(wrapper.text()).toContain('สวัสดีตอนเช้า');
  });

  it('shows สวัสดีตอนบ่าย between 12:00 and 18:00', () => {
    const date = new Date(2025, 0, 1, 14, 0, 0);
    vi.setSystemTime(date);
    
    const wrapper = mount(AppHeader);
    expect(wrapper.text()).toContain('สวัสดีตอนบ่าย');
  });

  it('shows สวัสดีตอนเย็น after 18:00', () => {
    const date = new Date(2025, 0, 1, 20, 0, 0);
    vi.setSystemTime(date);
    
    const wrapper = mount(AppHeader);
    expect(wrapper.text()).toContain('สวัสดีตอนเย็น');
  });
});
```

---

### Task 5: Test Router Logic (`router/index.ts`)

**Impact:** Medium. Ensures route guards and navigation work as expected.

1.  Create file: `tests/router/index.spec.ts`.
2.  Test route definitions and meta properties.

**Implementation:**

```typescript
import { describe, it, expect } from 'vitest';
import { createRouter, createWebHistory } from 'vue-router';
import routes from '@/router/index';

// Re-create router for testing to ensure clean state
const router = createRouter({
  history: createWebHistory(),
  routes: routes.map(r => ({ ...r, component: { template: '<div/>' } })), // Mock component
});

describe('Router', () => {
  it('defines the home route', () => {
    const homeRoute = routes.find(r => r.path === '/');
    expect(homeRoute).toBeDefined();
    expect(homeRoute?.meta?.tab).toBe('all');
  });

  it('defines the tools route', () => {
    const toolsRoute = routes.find(r => r.path === '/tools');
    expect(toolsRoute).toBeDefined();
    expect(toolsRoute?.meta?.tab).toBe('tool');
  });

  it('defines the reports route', () => {
    const reportsRoute = routes.find(r => r.path === '/reports');
    expect(reportsRoute).toBeDefined();
    expect(reportsRoute?.meta?.tab).toBe('report');
  });
  
  it('defines the external route', () => {
    const externalRoute = routes.find(r => r.path === '/external');
    expect(externalRoute).toBeDefined();
    expect(externalRoute?.meta?.tab).toBe('external');
  });
});
```

---

## ✅ Verification Protocol

After writing all test files, execute the following commands in sequence:

1.  **Run Tests:**
    ```bash
    bun run test:unit
    ```
    *Result:* All tests must pass (`PASS`).

2.  **Generate Coverage:**
    ```bash
    bun run test:coverage
    ```

3.  **Check Coverage Report:**
    - Look at the output in the terminal.
    - Ensure:
        - `statements` >= 70%
        - `branches` >= 70%
        - `functions` >= 70%
        - `lines` >= 70%
    - Open `coverage/index.html` in a browser to visualize which lines are missed. (Focus on `src/components/` and `src/stores/`).

4.  **Sanity Check (Optional but Recommended):**
    - Run `bun run build` to ensure tests didn't inadvertently mock types wrong (TypeScript check).

**Stop Condition:** Do not consider this mission complete until the console output shows at least **70%** coverage across the board.