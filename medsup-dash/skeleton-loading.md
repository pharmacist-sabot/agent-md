# Implement Skeleton Loading for Improved UX

## Context
The current loading state in `OverviewView.vue` displays a single centered `<Loader2 />` spinner (`.animate-spin`). This blocks the user from understanding the structure of the incoming data.

To improve the User Experience (UX) and Perceived Performance, we should implement **Skeleton Loading**. Skeletons provide a gray visual placeholder that mimics the actual layout of the page (cards, table rows) while data is being fetched.

## Objective
1.  Create a reusable `BaseSkeleton` component.
2.  Refactor `OverviewView.vue` to replace the spinner with a skeletal layout that mirrors the KPI cards, quarterly summary, and data table.

---

## Instructions for the AI Agent

### Step 1: Create the Base Skeleton Component
Create a new file `src/components/common/BaseSkeleton.vue`.

**Requirements:**
-   A simple component that renders a pulsing gray box.
-   Accept a `class` prop to allow flexibility in width, height, and border-radius.

**Code to Add:**

```vue
<script setup lang="ts">
defineProps<{
  class?: string;
}>();
</script>

<template>
  <div
    class="animate-pulse bg-gray-200 rounded"
    :class="class"
    aria-hidden="true"
  />
</template>
```

### Step 2: Refactor `OverviewView.vue`

Open `src/views/dashboard/OverviewView.vue`.

**1. Import the new component:**
```typescript
import BaseSkeleton from '@/components/common/BaseSkeleton.vue';
```

**2. Locate the Loading Block:**
Find the section:
```vue
<div v-if="store.loading" class="...">
  <Loader2 ... />
</div>
```

**3. Replace it with a Skeletal Layout:**
Replace the entire `<div v-if="store.loading">` block with the following structure. This must strictly mimic the layout of the real content (Grid of 3, Grid of 4, Table rows) to prevent layout shifts when loading finishes.

**New Loading Block Code:**

```vue
<!-- Loading State: Skeletons -->
<div v-if="store.loading" class="space-y-8 pb-12">
  <!-- Header Skeleton -->
  <div class="h-32 bg-white/40 backdrop-blur-sm p-6 rounded-2xl border border-white/50 flex items-center gap-4">
    <BaseSkeleton class="w-16 h-16 rounded-xl" />
    <div class="space-y-2 w-1/3">
      <BaseSkeleton class="h-6 w-48 rounded" />
      <BaseSkeleton class="h-4 w-64 rounded" />
    </div>
  </div>

  <!-- KPI Cards Skeleton (3 Grid) -->
  <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
    <div v-for="i in 3" :key="i" class="p-6 bg-white rounded-xl border border-gray-100 h-32">
      <div class="flex justify-between mb-4">
        <BaseSkeleton class="w-12 h-12 rounded-lg" />
        <BaseSkeleton class="w-8 h-8 rounded-full" />
      </div>
      <BaseSkeleton class="h-4 w-24 rounded mb-2" />
      <BaseSkeleton class="h-8 w-32 rounded" />
    </div>
  </div>

  <!-- Quarterly Report Skeleton (4 Grid) -->
  <div class="bg-white/80 backdrop-blur rounded-2xl border border-purple-100 shadow-sm p-6">
    <h3 class="text-lg font-bold text-gray-800 mb-6 flex items-center gap-2">
      <span class="w-1.5 h-6 bg-brand rounded-full" />
      <BaseSkeleton class="h-6 w-48 rounded" />
    </h3>
    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
      <div v-for="i in 4" :key="i" class="bg-purple-50/50 p-5 rounded-xl h-24">
        <BaseSkeleton class="h-4 w-16 rounded mb-2" />
        <BaseSkeleton class="h-6 w-24 rounded" />
      </div>
    </div>
  </div>

  <!-- Table Skeleton -->
  <div class="bg-white/80 backdrop-blur rounded-2xl border border-purple-100 shadow-sm p-6">
    <div class="flex justify-between items-center mb-5">
      <BaseSkeleton class="h-6 w-32 rounded" />
      <BaseSkeleton class="h-8 w-24 rounded" />
    </div>
    <div class="space-y-4">
      <!-- Table Header -->
      <div class="flex gap-4 pb-2 border-b border-gray-100">
        <BaseSkeleton class="h-6 w-24 rounded" />
        <BaseSkeleton class="h-6 w-32 rounded" />
        <BaseSkeleton class="h-6 w-32 rounded" />
        <BaseSkeleton class="h-6 w-24 rounded ml-auto" />
      </div>
      <!-- Table Rows (5 Rows) -->
      <div v-for="i in 5" :key="i" class="flex gap-4 items-center py-3">
        <BaseSkeleton class="h-5 w-24 rounded" />
        <BaseSkeleton class="h-5 w-32 rounded" />
        <BaseSkeleton class="h-6 w-24 rounded-full" />
        <BaseSkeleton class="h-5 w-24 rounded ml-auto" />
      </div>
    </div>
  </div>
</div>
```

### Step 3: Cleanup
Remove the `<Loader2 />` import from `<script setup>` if it is no longer needed anywhere else in the file (or leave it if used elsewhere).

---

## Validation

1.  Run `bun run dev`.
2.  Refresh the Dashboard page.
3.  Observe the **Skeleton** animation. It should look exactly like the real dashboard but grayed out and pulsing.
4.  When data finishes loading (`store.loading` becomes false), the skeletons should instantly snap into the real data layout.
5.  Check that there are no console errors about missing props or components.