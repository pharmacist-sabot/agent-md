# Progressive Web App (PWA)

> **Role:** PWA Engineer / Mobile UX Specialist  
> **Objective:** Convert the application into a PWA to enable offline capabilities, installability (Add to Home Screen), and improved performance.  
> **Strict Constraint:** Do not break the build or CSP. Ensure color branding matches `sabot-600`.

---

## 🎯 PWA Requirements Checklist

- [x] **Installability:** Browser displays "Install App" prompt.
- [x] **Offline Support:** Application works without internet (at least the shell + cached assets).
- [x] **Branding:** App icon and theme color match Sabot Hospital branding.
- [x] **Performance:** Service Worker caches static assets (JS, CSS, Images).

---

## 🛠 Mission Tasks

### Task 1: Install Dependencies

**Action:** Add the official Vite PWA plugin.

```bash
bun add -D vite-plugin-pwa
```

---

### Task 2: Configure Vite Plugin

**Action:** Update `vite.config.ts` to enable the Service Worker.

1.  Open `vite.config.ts`.
2.  Import and add `VitePWA` plugin.

**Implementation:**

```typescript
import tailwindcss from '@tailwindcss/vite';
import vue from '@vitejs/plugin-vue';
import { fileURLToPath, URL } from 'node:url';
import { defineConfig } from 'vite';
import { VitePWA } from 'vite-plugin-pwa'; // <--- ADD IMPORT

export default defineConfig({
  plugins: [
    vue(),
    tailwindcss(),
    // ADD THIS CONFIGURATION
    VitePWA({
      registerType: 'autoUpdate',
      includeAssets: ['favicon.ico', 'robots.txt', 'apple-touch-icon.png'],
      manifest: {
        name: 'Pharmacy Hub Sabot',
        short_name: 'PharmacyHub',
        description: 'ระบบเครื่องมือและบริการดิจิทัลเภสัชกรรม โรงพยาบาลสระโบสถ์',
        theme_color: '#5d8736', // Corresponds to --color-sabot-600
        background_color: '#eeefe0', // Corresponds to --color-sabot-100
        display: 'standalone',
        orientation: 'portrait',
        scope: '/',
        start_url: '/',
        icons: [
          {
            src: 'pwa-192x192.png',
            sizes: '192x192',
            type: 'image/png',
          },
          {
            src: 'pwa-512x512.png',
            sizes: '512x512',
            type: 'image/png',
          },
          {
            src: 'pwa-512x512.png',
            sizes: '512x512',
            type: 'image/png',
            purpose: 'any maskable',
          },
        ],
      },
      // Cache Strategy: NetworkFirst for navigation, StaleWhileRevalidate for assets
      workbox: {
        globPatterns: ['**/*.{js,css,html,ico,png,svg,json,woff2}'],
        runtimeCaching: [
          {
            urlPattern: /^https:\/\/script\.google\.com\/.*/i,
            handler: 'NetworkFirst',
            options: {
              cacheName: 'google-scripts-cache',
              expiration: {
                maxEntries: 10,
                maxAgeSeconds: 60 * 60 * 24, // 24 hours
              },
            },
          },
          {
            urlPattern: ({ request }) => request.destination === 'image',
            handler: 'CacheFirst',
            options: {
              cacheName: 'images-cache',
              expiration: {
                maxEntries: 60,
                maxAgeSeconds: 30 * 24 * 60 * 60, // 30 Days
              },
            },
          },
        ],
      },
    }),
  ],
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url)),
    },
  },
});
```

---

### Task 3: Create App Icons

**Action:** Generate or place app icons in the `public/` folder.

**Instructions:**
1.  You need to create (or ask a designer for) two images:
    -   `public/pwa-192x192.png` (Size 192x192 pixels)
    -   `public/pwa-512x512.png` (Size 512x512 pixels)
2.  *Design Guideline:* Use the `Pill` icon from the app on a solid background color `#5d8736` (Sabot-600) with a white stroke.

*(Note: If you don't have icons ready, the build will warn you, but the app will still work. For "World-Class" standards, valid icons are required.)*

---

### Task 4: Update HTML Meta Tags

**Action:** Enhance `index.html` for Apple and PWA specific rendering.

**Implementation:**

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    
    <!-- PWA Theme Color -->
    <meta name="theme-color" content="#5d8736" />
    
    <!-- Apple Touch Icon -->
    <link rel="apple-touch-icon" href="/apple-touch-icon.png" />
    
    <!-- Fonts -->
    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
    <link href="https://fonts.googleapis.com/css2?family=Prompt:wght@300;400;500;700&display=swap" rel="stylesheet" />
    
    <title>Pharmacy Hub Sabot</title>
  </head>
  <body>
    <div id="app"></div>
    <script type="module" src="/src/main.ts"></script>
  </body>
</html>
```

---

### Task 5: Create Install Prompt Component (UX)

**Action:** Create a "Install App" button that appears when the browser suggests installation.

1.  Create file: `src/components/layout/PwaInstallPrompt.vue`.
2.  Implement logic using `virtual:pwa-register/vue`.

**Implementation:**

```vue
<script setup lang="ts">
import { useRegisterSW } from 'virtual:pwa-register/vue';
import { ref, watchEffect } from 'vue';

const {
  offlineReady,
  needRefresh,
  updateSW,
} = useRegisterSW();

const offlineReadyClose = () => {
  offlineReady.value = false;
};

const refreshApp = () => {
  updateSW(true);
};
</script>

<template>
  <div
    v-if="offlineReady || needRefresh"
    class="fixed bottom-4 right-4 z-50 bg-white p-4 rounded-2xl shadow-2xl border border-sabot-200 flex flex-col gap-3 max-w-xs"
  >
    <div class="text-sm font-medium text-gray-700">
      <div v-if="offlineReady">
        ✅ App is ready to work offline.
      </div>
      <div v-else>
        🔄 New content available, click on reload button to update.
      </div>
    </div>
    
    <div class="flex gap-2 justify-end">
      <button
        v-if="offlineReady"
        @click="offlineReadyClose"
        class="px-3 py-1 text-sm font-bold text-gray-500 hover:text-gray-700"
      >
        Close
      </button>
      <button
        v-else
        @click="refreshApp"
        class="px-3 py-1 text-sm font-bold bg-sabot-600 text-white rounded-lg hover:bg-sabot-700 transition-colors"
      >
        Reload
      </button>
    </div>
  </div>
</template>
```

3.  **Import into App:**
    Open `src/App.vue` and add the component.

```vue
<!-- src/App.vue -->
<script setup lang="ts">
import { computed } from 'vue';
import { RouterView, useRoute } from 'vue-router';
import BlankLayout from '@/layouts/BlankLayout.vue';
import DefaultLayout from '@/layouts/DefaultLayout.vue';
// ADD THIS IMPORT
import PwaInstallPrompt from '@/components/layout/PwaInstallPrompt.vue'; 

const route = useRoute();

// Mapping layout names to components
const layouts = {
  default: DefaultLayout,
  blank: BlankLayout,
};

const currentLayout = computed(() => {
  const layoutName = (route.meta.layout as keyof typeof layouts) || 'default';
  return layouts[layoutName];
});
</script>

<template>
  <component :is="currentLayout">
    <RouterView />
  </component>
  
  <!-- ADD THIS COMPONENT HERE -->
  <PwaInstallPrompt />
</template>
```

---

### Task 6: Update CSP Headers (Critical)

**Action:** Service Workers require specific permissions in the Content Security Policy.

1.  Open `firebase.json`.
2.  Update the `Content-Security-Policy` header.

**Update this line:**
```json
"value": "default-src 'self'; script-src 'self'; ..."
```
**To this:**
```json
"value": "default-src 'self'; script-src 'self' 'unsafe-inline' https://fonts.googleapis.com; worker-src 'self' blob:; ..."
```

*(Full updated `firebase.json` recommendation):*
```json
{
  "hosting": {
    "public": "dist",
    "ignore": ["firebase.json", "**/.*", "**/node_modules/**"],
    "rewrites": [{ "source": "**", "destination": "/index.html" }],
    "headers": [
      {
        "source": "**",
        "headers": [
          {
            "key": "Content-Security-Policy",
            "value": "default-src 'self'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src 'self' https://fonts.gstatic.com; img-src 'self' data: blob:; script-src 'self'; connect-src 'self' https://script.google.com https://*.web.app blob:; worker-src 'self' blob:;"
          },
          {
            "key": "Cache-Control",
            "value": "max-age=1800"
          }
        ]
      },
      {
        "source": "**/*.@(jpg|jpeg|gif|png|webp|svg)",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "max-age=604800"
          }
        ]
      }
    ]
  }
}
```

---

## ✅ Verification Protocol

1.  **Build Test:**
    ```bash
    bun run build
    ```
    *Result:* Build must succeed. Check console for `PWA v0.x.x building...`.

2.  **Preview Test:**
    ```bash
    bun run preview
    ```
    *Note:* PWA features (Service Worker) might not fully register in `vite preview` in some environments, but the files should be generated in `dist/sw.js`.

3.  **Lighthouse Audit (The Gold Standard):**
    - Open Chrome DevTools on the `preview` or hosted site.
    - Go to the **Lighthouse** tab.
    - Select **Progressive Web App**.
    - Click **Analyze page load**.

4.  **Target Metrics:**
    - **Installable:** Yes (Show a blue installable badge ✅).
    - **PWA Optimized:** Score > 90.
    - **Manifest exists:** ✅.

**Stop Condition:** If Lighthouse fails "Installable" check, usually missing icons or start_url mismatch.