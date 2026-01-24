# Security & Type Safety Enforcement

> **Role:** Senior Frontend Engineer / TypeScript Expert  
> **Objective:** Execute critical refactoring tasks with zero regression, strictly adhering to "World-Class" standards.

---

## 🚀 Mission Overview

This document contains instructions to resolve critical technical debt and security vulnerabilities. Follow each step meticulously. Do not improvise outside the scope of this mission.

---

## 📋 Mission Tasks

### Task 1: Fix `iconMap` Type Safety in `ResourceCard.vue`

**Severity:** High (Type Safety Violation)  
**Impact:** Prevents TypeScript from catching invalid component assignments.

**Instructions:**
1.  Open `src/components/common/ResourceCard.vue`.
2.  Locate the `iconMap` constant definition inside `<script setup>`.
3.  Modify the type annotation from `any` to `Record<string, Component>`.
4.  Ensure `Component` is imported from `vue`.
5.  **Do NOT** change the object structure or keys, only the type definition.

**Implementation Example:**

```vue
<script setup lang="ts">
import { Component } from 'vue'; // <--- ADD THIS IMPORT
import { computed } from 'vue';
import type { ResourceItem } from '@/types/resource';
// ... other imports

const props = defineProps<{ item: ResourceItem }>();

// CHANGE THIS:
// const iconMap: Record<string, any> = { ... }

// TO THIS:
const iconMap: Record<string, Component> = {
  AlertTriangle,
  FileSignature,
  Calculator,
  Baby,
  FileDown,
  Pill,
  Siren,
  ClipboardList,
  BarChart3,
  PieChart,
  CalendarRange,
  Banknote,
  Users,
};
</script>
```

**Verification:**
- Run `bun run type-check`. Output must show "0 errors".
- Run `bun run lint`. No errors regarding `ts/no-explicit-any` should exist in this file.

---

### Task 2: Audit & Remove Unused Dependency (`axios`)

**Severity:** Medium (Performance & Bundle Size)  
**Impact:** Reduces production bundle size by ~13kb (gzipped) and reduces attack surface.

**Instructions:**
1.  Perform a global search (grep) for the string `axios` in the `src/` directory.
2.  **Search Query:** Check for imports like `import axios from 'axios'`, `import axios from 'axios/dist/axios'`, or usage like `axios.get()`.
3.  **Confirmation:** If NO usage is found in `src/` (excluding `node_modules` or `dist`), proceed to removal.
    *   *Note:* This project uses External Links (`href` with `target="_blank"`), so `axios` is likely dead code.
4.  Remove the package using the package manager:
    ```bash
    bun remove axios
    ```
5.  Verify removal in `package.json`.

**Verification:**
- Run `bun run build`. The build MUST complete successfully.
- Run `bun run type-check`. Ensure no "Cannot find module 'axios'" errors appear in `src/`.

---

### Task 3: Implement Content Security Policy (CSP) in `firebase.json`

**Severity:** Critical (Security Vulnerability)  
**Impact:** Protects users from XSS (Cross-Site Scripting) attacks.

**Instructions:**
1.  Open `firebase.json`.
2.  Add a `headers` object within the `"hosting"` configuration.
3.  Define a `Content-Security-Policy` header that allows only necessary sources.

**Security Policy Requirements:**
- Allow fonts from Google Fonts (`fonts.googleapis.com`, `fonts.gstatic.com`).
- Allow scripts and styles from self only.
- Allow images/data for internal usage.
- Block mixed content (upgrade insecure requests).

**Implementation Example:**

```json
{
  "hosting": {
    "public": "dist",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ],
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ],
    "headers": [
      {
        "source": "**",
        "headers": [
          {
            "key": "Content-Security-Policy",
            "value": "default-src 'self'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src 'self' https://fonts.gstatic.com; img-src 'self' data: blob:; script-src 'self'; connect-src 'self' https://script.google.com https://*.web.app;"
          },
          {
            "key": "X-Content-Type-Options",
            "value": "nosniff"
          },
          {
            "key": "X-Frame-Options",
            "value": "DENY"
          },
          {
            "key": "Referrer-Policy",
            "value": "strict-origin-when-cross-origin"
          }
        ]
      }
    ]
  }
}
```

**Verification:**
- Run `bun run build`.
- Run `bun run preview` (or deploy to staging).
- Open Browser DevTools (F12) -> Console. There should be NO CSP violations.

---

## ✅ Post-Mission Checklist

Before marking this mission as complete, run the full suite of quality gates:

1.  **Type Check:** `bun run type-check` (Must pass)
2.  **Linter:** `bun run lint` (Must pass)
3.  **Build:** `bun run build` (Must generate `dist/` successfully)
4.  **Preview:** `bun run preview` (Visit localhost:4173, verify Homepage loads and Icons display correctly)

**Failure to pass any of the above gates requires an immediate rollback (git reset).**