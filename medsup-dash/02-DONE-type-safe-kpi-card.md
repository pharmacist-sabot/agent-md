# Type-Safe KPI Card Icon Prop

## Context
The `KPICard.vue` component currently uses `icon?: any` for its prop. This violates the project's strict TypeScript configuration and disables type checking for one of the component's most critical visual elements. Passing a string, object, or invalid component would cause runtime errors.

## Objective
Refactor the `icon` prop type to use `DefineComponent` from `vue`. This ensures that only valid Vue components (like Lucide icons) can be passed to the card.

---

## Instructions for the AI Agent

### Step 1: Update Imports and Types in `KPICard.vue`

Open `src/components/dashboard/KpiCard.vue`.

**Requirements:**
1.  Import `DefineComponent` from `'vue'`.
2.  Update the `Props` interface to strictly type the `icon` property.

**Change from:**

```typescript
<script setup lang="ts">
defineProps<{
  title: string;
  value: string;
  subValue?: string;
  icon?: any; // <--- Vague type, unsafe
  colorClass?: string;
}>();
</script>
```

**Change to:**

```typescript
<script setup lang="ts">
import type { DefineComponent } from 'vue';

defineProps<{
  title: string;
  value: string;
  subValue?: string;
  icon?: DefineComponent; // <--- Strict type for Vue component definitions
  colorClass?: string;
}>();
</script>
```

**Note:** Since this is a `script setup` file using generic `defineProps`, TypeScript will automatically infer the props.

### Step 2: Verify Usage in Parent (Optional but Recommended Check)

Although not strictly necessary for the refactor to work, it is good practice to glance at `src/views/dashboard/OverviewView.vue`.
Ensure that the icons being imported from `lucide-vue-next` (e.g., `Coins`, `Receipt`) are indeed compatible. Most modern icon libraries export standard Vue components that extend `DefineComponent`, so this should match perfectly without errors.

**Example of usage (No changes needed here, just validation):**
```vue
<KpiCard
  :icon="Coins"
  ... />
```

---

## Validation

After applying this change:
1.  Run `bun run type-check`.
2.  Verify that **no** type errors occur.
3.  If you were to temporarily pass a string like `icon="hello"` to `KPICard` in the code, TypeScript should now raise a red squiggly line error (Type 'string' is not assignable to type 'DefineComponent').
4.  The application should build and run exactly as before, with the added safety of type checking.