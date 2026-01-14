# TypeScript Migration & Type Safety Implementation

**Objective:**  
Migrate the codebase from JavaScript (`.js`) to TypeScript (`.ts`) and introduce strongly typed Interfaces for Drug models and API responses.
**Goal:**  
Achieve 100% type safety to prevent runtime errors, improve developer experience (DX), and ensure code reliability at scale.
**Standard:**  
Vue 3 + Vite + TypeScript (Strict Mode) Best Practices.

---

## 🛑 PRE-REQUISITES: Backup & Safety

**CRITICAL:** Before starting any code changes:
1.  **Commit all current changes** to Git.
2.  Create a new branch: `git checkout -b feature/typescript-migration`.
3.  **Do NOT delete original `.js` files immediately.** Rename them (e.g., `drugStore.js` -> `drugStore.ts`) so you can revert easily if necessary.

---

## 1. Phase I: Dependency Installation

Install the necessary TypeScript tooling and type definitions for Node.js.

**Action:**
```bash
npm install -D typescript vue-tsc @types/node
```

---

## 2. Phase II: TypeScript Configuration

Create the configuration file that instructs Vite and the compiler how to handle TypeScript.

**New File:** `tsconfig.json` (Root Directory)

**Instructions:**
1.  Create `tsconfig.json`.
2.  Use the configuration below optimized for Vue 3 Vite projects.
3.  **Note:** `strict: true` is enabled. This is a world-class standard. If you see errors, fix the types, do not disable strict mode.

**Code Content:**
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "skipLibCheck": true,

    /* Bundler mode */
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "preserve",

    /* Linting (STRICT MODE) */
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,

    /* Path Mapping */
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src/**/*.ts", "src/**/*.tsx", "src/**/*.vue"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

**New File:** `tsconfig.node.json` (Root Directory)

**Instructions:**
1.  Create `tsconfig.node.json` to support Vite config typing.

**Code Content:**
```json
{
  "compilerOptions": {
    "composite": true,
    "skipLibCheck": true,
    "module": "ESNext",
    "moduleResolution": "bundler",
    "allowSyntheticDefaultImports": true,
    "strict": true
  },
  "include": ["vite.config.ts"]
}
```

---

## 3. Phase III: Build Configuration Migration

Update the Vite configuration file to use TypeScript.

**File:** `vite.config.js` -> Rename to `vite.config.ts`

**Instructions:**
1.  Rename `vite.config.js` to `vite.config.ts`.
2.  Update the code to match the TypeScript version of imports.

**Code Content:**
```typescript
import { fileURLToPath, URL } from 'node:url'
import { defineConfig, loadEnv } from 'vite'
import vue from '@vitejs/plugin-vue'
import { VitePWA } from 'vite-plugin-pwa'

// https://vitejs.dev/config/
export default defineConfig(({ mode }) => {
  // TypeScript will infer the correct types for env
  const env = loadEnv(mode, process.cwd(), '')

  return {
    plugins: [
      vue(),
      VitePWA({
        registerType: 'autoUpdate',
        includeAssets: ['favicon.ico', 'apple-touch-icon.png', 'masked-icon.svg'],
        manifest: {
          name: 'High Alert Drugs รพ.สระโบสถ์',
          short_name: 'HAD-SBH',
          description: 'คู่มือข้อมูลยาความเสี่ยงสูงสำหรับบุคลากรทางการแพทย์ โรงพยาบาลสระโบสถ์',
          theme_color: '#0066b3',
          background_color: '#f7fafe',
          display: 'standalone',
          scope: '/',
          start_url: '/',
          icons: [
            {
              src: 'pwa-192x192.png',
              sizes: '192x192',
              type: 'image/png'
            },
            {
              src: 'pwa-512x512.png',
              sizes: '512x512',
              type: 'image/png'
            }
          ]
        },
        workbox: {
          globPatterns: ['**/*.{js,css,html,ico,png,svg,woff,woff2}'],
          runtimeCaching: [
            {
              urlPattern: ({ url }) => url.origin === env.VITE_SUPABASE_URL,
              handler: 'StaleWhileRevalidate',
              options: {
                cacheName: 'supabase-api-cache',
                expiration: {
                  maxEntries: 50,
                  maxAgeSeconds: 60 * 60 * 24 * 7 // 7 days
                },
                cacheableResponse: {
                  statuses: [0, 200]
                }
              }
            }
          ]
        },
        devOptions: {
          enabled: true
        }
      })
    ],
    resolve: {
      alias: {
        '@': fileURLToPath(new URL('./src', import.meta.url))
      }
    }
  }
})
```

---

## 4. Phase IV: Type Definitions (The Model Layer)

Create centralized interfaces to represent the data structure. This is the core of the "Data Model" request.

**New File:** `src/types/drug.ts`

**Instructions:**
1.  Create directory `src/types/` if it doesn't exist.
2.  Define the `Drug` interface based on the Supabase table structure.
3.  **NOTE:** Review the Supabase table columns and match them exactly here. Common fields are provided below as a template.

**Code Content:**
```typescript
/**
 * Represents a High-Alert Drug entity.
 * Maps to the 'drugs' table in Supabase.
 */
export interface Drug {
  id: string
  created_at: string
  
  // Core identifiers
  generic_name: string
  trade_name: string | null // Nullable if not all drugs have a trade name
  
  // Clinical Details (Add/Remove fields based on your actual DB schema)
  description?: string
  dosage_and_administration?: string
  side_effects?: string
  monitoring_parameters?: string
  high_alert_reason?: string
  
  // Metadata
  category?: string
}

/**
 * Represents the API error structure returned by Supabase
 */
export interface ApiError {
  message: string
  details: string | null
  hint: string | null
  code: string
}
```

---

## 5. Phase V: Logic & Utility Migration

### A. Supabase Client

**File:** `src/lib/supabaseClient.js` -> Rename to `src/lib/supabaseClient.ts`

**Instructions:**
1.  Rename to `.ts`.
2.  Ensure it exports the client correctly. No major logic changes usually needed here unless adding type generics later.

### B. Pinia Store (CRITICAL STEP)

**File:** `src/stores/drugStore.js` -> Rename to `src/stores/drugStore.ts`

**Instructions:**
1.  Rename to `.ts`.
2.  Import the `Drug` interface.
3.  Explicitly type the `ref`s.

**Code Content:**
```typescript
import { ref, computed } from 'vue'
import { defineStore } from 'pinia'
import { supabase } from '../lib/supabaseClient'
import type { Drug } from '../types/drug' // 👈 Import Interface

export const useDrugStore = defineStore('drug', () => {
  // State (Explicitly typed)
  const drugs = ref<Drug[]>([])
  const searchQuery = ref<string>('')
  const selectedDrug = ref<Drug | null>(null)
  const isLoading = ref<boolean>(false)
  const error = ref<string | null>(null)

  // Getters
  const hasResults = computed(() => drugs.value.length > 0)
  const isEmpty = computed(() => !isLoading.value && !hasResults.value && searchQuery.value.length > 0)

  // Actions
  const fetchDrugs = async (query: string) => {
    if (!query || query.trim() === '') {
      drugs.value = []
      return
    }

    searchQuery.value = query
    isLoading.value = true
    error.value = null

    try {
      // Supabase typing can be enhanced later, but this returns 'any' by default in simple setups
      // We will cast the result to Drug[] to ensure type safety in the store usage
      const { data, error: fetchError } = await supabase
        .from('drugs')
        .select('*')
        .or(`generic_name.ilike.%${query}%,trade_name.ilike.%${query}%`)
        .limit(50)

      if (fetchError) throw fetchError

      drugs.value = (data as Drug[]) || []
    } catch (err: any) {
      console.error('Error fetching drugs:', err)
      error.value = err.message
      drugs.value = []
    } finally {
      isLoading.value = false
    }
  }

  const selectDrug = (drug: Drug) => {
    selectedDrug.value = drug
  }

  const clearSelection = () => {
    selectedDrug.value = null
  }

  const resetSearch = () => {
    drugs.value = []
    searchQuery.value = ''
    error.value = null
    isLoading.value = false
  }

  return {
    drugs,
    searchQuery,
    selectedDrug,
    isLoading,
    error,
    hasResults,
    isEmpty,
    fetchDrugs,
    selectDrug,
    clearSelection,
    resetSearch
  }
})
```

---

## 6. Phase VI: Component Migration

### A. Root Application

**File:** `src/main.js` -> Rename to `src/main.ts`

**Instructions:**
1.  Rename file.
2.  Update `index.html` script source: `<script type="module" src="/src/main.ts"></script>` (Vite usually handles this automatically if configured, but check to be sure).

### B. Vue Components (`App.vue`, `DrugSearch.vue`, etc.)

**Instructions:**
1.  Open each `.vue` file.
2.  Change `<script setup>` to `<script setup lang="ts">`.
3.  **Props Typing:** Replace simple arrays with `defineProps`.
    *   *Before:* `const props = defineProps(['drug'])`
    *   *After:*
        ```typescript
        interface Props {
          drug: Drug
        }
        const props = defineProps<Props>()
        ```
4.  **Emits Typing:**
    *   *Before:* `const emit = defineEmits(['close'])`
    *   *After:*
        ```typescript
        const emit = defineEmits<{
          (e: 'close'): void
          (e: 'select', drug: Drug): void
        }>()
        ```

---

## 7. Phase VII: Verification & Error Handling

1.  **Run Type Checker:**
    ```bash
    npm run type-check
    ```
    *(Note: Add `"type-check": "vue-tsc --noEmit"` to scripts in package.json if not present).*

2.  **Fix Type Errors:**
    *   The agent will likely find errors due to `unknown` types or mismatched interfaces.
    *   **Rule:** Fix the Interface definition first, then fix the component implementation. Never use `as any` unless absolutely necessary for a 3rd party library.

3.  **Run Dev Server:**
    ```bash
    npm run dev
    ```
    Ensure the application runs exactly as it did before.

---

## 8. Summary of Actions

1.  Install `typescript`, `vue-tsc`.
2.  Create `tsconfig.json` and `tsconfig.node.json`.
3.  Rename `vite.config.js` -> `vite.config.ts`.
4.  Create `src/types/drug.ts` with the `Drug` interface.
5.  Migrate `src/lib/supabaseClient.js` -> `.ts`.
6.  Migrate `src/stores/drugStore.js` -> `.ts` and apply `ref<Drug[]>()`.
7.  Migrate `src/main.js` -> `.ts`.
8.  Update all `.vue` components to `lang="ts"` and type props/emit.