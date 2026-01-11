# Task: Setup Offline Support (PWA) with Service Worker

## Objective
Transform the web application into a Progressive Web App (PWA) to ensure 100% offline functionality. This is critical for a Medical App where users might be in low-signal areas (hospitals, clinics).

**Requirements:**
1.  **Caching Strategy:** Cache the WASM module, JS bundle, CSS, and HTML to enable full functionality offline.
2.  **Manifest:** Add a Web App Manifest so users can "Add to Home Screen" (Installable App).
3.  **Updates:** Prompt the user when a new version is available (Don't auto-reload while the user is calculating a dose).

---

## Phase 1: Installation

Install the official Vite PWA plugin:

```bash
npm install -D vite-plugin-pwa workbox-window
```

---

## Phase 2: Configuration (`vite.config.ts`)

Modify `vite.config.ts` to register the PWA plugin. The configuration below uses the **StaleWhileRevalidate** strategy, providing the fastest possible load times while ensuring updates are fetched in the background.

**Crucial for WASM:** Ensure the WASM file is explicitly included in the assets to cache.

```typescript
import { fileURLToPath, URL } from 'node:url'
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import vueDevTools from 'vite-plugin-vue-devtools'
import wasm from 'vite-plugin-wasm'
import { VitePWA } from 'vite-plugin-pwa' // Import

// https://vite.dev/config/
export default defineConfig({
  plugins: [
    vue(),
    vueDevTools(),
    wasm(),
    
    // PWA Configuration
    VitePWA({
      registerType: 'autoUpdate', // Automatically update SW, we handle UI prompts manually
      includeAssets: ['**/*.wasm'], // FORCE CACHE THE RUST WASM MODULE
      manifest: {
        name: 'Warfarin Calculator',
        short_name: 'Warfarin',
        description: 'Precision Dosing Assistant',
        theme_color: '#ffffff',
        background_color: '#F5F5F7',
        display: 'standalone',
        icons: [
          {
            src: 'https://picsum.photos/seed/warfarin-icon/192/192', // Placeholder - Replace with real assets
            sizes: '192x192',
            type: 'image/png'
          },
          {
            src: 'https://picsum.photos/seed/warfarin-icon/512/512', // Placeholder - Replace with real assets
            sizes: '512x512',
            type: 'image/png'
          }
        ]
      },
      workbox: {
        // Precache critical files
        globPatterns: ['**/*.{js,css,html,ico,png,svg,wasm}'],
        // Runtime caching for specific behaviors if needed
        runtimeCaching: [{
          urlPattern: /^https:\/\/fonts\.googleapis\.com\/.*/i,
          handler: 'CacheFirst',
          options: {
            cacheName: 'google-fonts-cache',
            expiration: {
              maxEntries: 10,
              maxAgeSeconds: 60 * 60 * 24 * 365 // <== 365 days
            },
            cacheableResponse: {
              statuses: [0, 200]
            }
          }
        }]
      }
    }),
  ],
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url))
    },
  },
})
```

---

## Phase 3: Update Notification UI

We need to tell the user when a new update is ready so they can refresh. We will create a composable and add a reload prompt to `App.vue`.

### 1. Create `src/composables/useUpdateSW.ts`

```typescript
import { registerSW } from 'virtual:pwa-register'

if ('serviceWorker' in navigator) {
  const updateSW = registerSW({
    onNeedRefresh() {
      // Trigger an event or show a custom modal
      const event = new CustomEvent('pwa-update-available')
      window.dispatchEvent(event)
    },
    onOfflineReady() {
      console.log('App is ready to work offline!')
    },
    onRegistered(registration) {
      // Periodically check for updates (every hour)
      setInterval(() => {
        registration && registration.update()
      }, 60 * 60 * 1000)
    },
  })
}
export default {}
```

### 2. Integrate into `src/main.ts`

Import the composable immediately to register the service worker:

```typescript
import { createApp } from 'vue'
import App from './App.vue'
import './composables/useUpdateSW' // <--- REGISTER SW HERE

createApp(App).mount('#app')
```

### 3. Add Reload UI in `App.vue`

Add a simple banner at the bottom of the screen that appears only when an update is ready.

```vue
<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const showUpdateBanner = ref(false)

const handleUpdate = () => {
  window.location.reload()
}

onMounted(() => {
  window.addEventListener('pwa-update-available', () => {
    showUpdateBanner.value = true
  })
})

onUnmounted(() => {
  window.removeEventListener('pwa-update-available', () => {})
})

// ... existing code
</script>

<template>
  <div id="app" class="relative min-h-screen">
    <!-- Existing App Content -->
    <!-- ... -->

    <!-- PWA Update Banner -->
    <div v-if="showUpdateBanner" class="fixed bottom-4 left-4 right-4 z-50 bg-blue-600 text-white px-4 py-3 rounded-xl shadow-lg flex items-center justify-between sm:w-auto sm:left-1/2 sm:-translate-x-1/2 sm:bottom-8 sm:min-w-[300px]">
      <div>
        <div class="font-bold text-sm">Update Available</div>
        <div class="text-xs text-blue-100">New version ready to install.</div>
      </div>
      <button @click="handleUpdate" class="bg-white text-blue-600 px-3 py-1 rounded-lg text-sm font-bold hover:bg-blue-50 transition">
        Reload
      </button>
    </div>
  </div>
</template>
```

---

## Phase 4: Verification & Testing

### 1. Local Testing
1.  Run `npm run build`.
2.  Run `npm run preview`.
3.  Open Chrome DevTools -> **Application** tab.
4.  Check **Service Workers**: You should see the service worker registered and active.
5.  Check **Manifest**: You should see the parsed manifest data.

### 2. Offline Simulation
1.  In the **Network** tab of DevTools, change throttling to **Offline**.
2.  Reload the page.
3.  **Result:** The app must load perfectly. The "Generate" button must calculate results (proving WASM is cached and running).

---

## Asset Management (Icons)
The configuration above uses placeholder images (`picsum.photos`). For a production-grade app:
1.  Create `public/icon-192x192.png` and `public/icon-512x512.png`.
2.  Update `vite.config.ts` `manifest.icons.src` paths to point to these real files.

## Validation Steps
1.  **Lighthouse Audit:** Run Chrome Lighthouse on the app. It should score 100 for "Progressive Web App" and "Offline Capabilities".
2.  **WASM Execution:** Verify that the app performs calculations while the device is in Airplane Mode.