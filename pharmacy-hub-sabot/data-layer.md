# Data Layer Decoupling

> **Role:** Backend/Frontend Integration Engineer  
> **Objective:** Decouple static data from TypeScript source code to external JSON files to enable hot-reloading configuration without recompilation.  
> **Strict Constraint:** Maintain 100% type safety during data fetching. Do not break existing filtering logic.

---

## 🎯 Architecture Overview

**Current State:**
- Data is hardcoded in `src/data/*.ts` files.
- Data is bundled into JavaScript at build time.

**Target State:**
- Data resides in `public/data/resources.json`.
- Data is fetched asynchronously via a Vue Composable (`useResources`).
- Loading and Error states are handled gracefully.

---

## 🛠 Mission Tasks

### Task 1: Create the JSON Data File

**Action:** Combine all data from `tools.ts`, `reports.ts`, and `externals.ts` into a single source of truth.

1.  Create directory: `public/data/`.
2.  Create file: `public/data/resources.json`.
3.  Paste the following content. *Ensure strict adherence to the structure (no trailing commas in JSON).*

**File Content (`public/data/resources.json`):**

```json
[
  {
    "id": "med-safety",
    "title": "MedSafety Net",
    "description": "ระบบบันทึกความคลาดเคลื่อนทางยา",
    "iconName": "AlertTriangle",
    "url": "https://script.google.com/macros/s/AKfycbyrXXZahRLu762PTxm72CtvF9UB9m6L5NNjdH7I06ARkSyJhBpc9O3fmse9SnHXR8Wi/exec",
    "isActive": true,
    "type": "tool"
  },
  {
    "id": "med-support",
    "title": "ระบบบันทึกมูลค่ายาสนับสนุน",
    "description": "บันทึกมูลค่ายาสนับสนุนเช่น ยา TB วัคซีน",
    "iconName": "FileSignature",
    "url": "https://med-support-record.web.app/",
    "isActive": true,
    "type": "tool"
  },
  {
    "id": "warfarin-calc",
    "title": "โปรแกรมคำนวณยา Warfarin",
    "description": "เครื่องมือช่วยคำนวณขนาดยา Warfarin",
    "iconName": "Calculator",
    "url": "https://sabot-warfarin-calculator.web.app/",
    "isActive": true,
    "type": "tool"
  },
  {
    "id": "pedi-dose",
    "title": "โปรแกรมคำนวณยาน้ำเด็ก",
    "description": "คำนวณขนาดยาน้ำสำหรับผู้ป่วยเด็กอย่างแม่นยำ",
    "iconName": "Baby",
    "url": "https://pedi-dose-c9cec.web.app/",
    "isActive": true,
    "type": "tool"
  },
  {
    "id": "doc-download",
    "title": "ระบบดาวน์โหลดเอกสาร",
    "description": "ดาวน์โหลดเอกสาร กลุ่มงานเภสัชกรรมฯ รพ.สระโบสถ์",
    "iconName": "FileDown",
    "url": "https://script.google.com/macros/s/AKfycbwA7S3gK4cNiwUdgi5FmD8Sh2A46kK-fHaZSjOHJVnTe-TvUAsSmygPjxUINPgMI3KI/exec",
    "isActive": true,
    "type": "tool"
  },
  {
    "id": "hospital-drugs",
    "title": "บัญชียาโรงพยาบาล",
    "description": "ตรวจสอบรายการยาที่มีใช้ในโรงพยาบาลสระโบสถ์",
    "iconName": "Pill",
    "url": "https://sabot-drug-lists.rxdevman.com",
    "isActive": true,
    "type": "tool"
  },
  {
    "id": "had-list",
    "title": "บัญชียา High-Alert Drugs",
    "description": "รายการยาที่ต้องใช้ความระมัดระวังเป็นพิเศษ (HAD)",
    "iconName": "Siren",
    "url": "https://high-alert-drugs-sabot.web.app/",
    "isActive": true,
    "type": "tool"
  },
  {
    "id": "drug-tracker",
    "title": "ระบบ DrugTracker",
    "description": "ระบบบันทึกและติดตามการสั่งซื้อยา",
    "iconName": "ClipboardList",
    "url": "https://drug-tracker-system.web.app/",
    "isActive": true,
    "type": "tool"
  },
  {
    "id": "e-lactancia",
    "title": "e-Lactancia",
    "description": "ฐานข้อมูลการใช้ยาในหญิงให้นมบุตร (ตรวจสอบความปลอดภัยของยา)",
    "iconName": "Link",
    "url": "https://www.e-lactancia.org/",
    "isActive": true,
    "type": "tool"
  },
  {
    "id": "thai-acc-warfarin",
    "title": "Warfarin and NOACs Registry Network",
    "description": "ระบบลงทะเบียนผู้ป่วย Warfarin และ NOACs แห่งประเทศไทย",
    "iconName": "Link",
    "url": "http://thaiacc.org/warfarin/",
    "isActive": true,
    "type": "external"
  },
  {
    "id": "dashboard-safety",
    "title": "MedSafety Net Dashboard",
    "description": "รายงานและจัดการคลาดเคลื่อนทางยา",
    "iconName": "BarChart3",
    "url": "https://script.google.com/macros/s/AKfycbytUM2FwmVTiTqTBjS_p5LEszr-X92tobG9LDLNsqjZug70wnwBazvKQBuu0PET4MUl/exec",
    "isActive": true,
    "type": "report"
  },
  {
    "id": "dashboard-support",
    "title": "รายงานมูลค่ายาสนับสนุน",
    "description": "รายงานมูลค่ายาสนับสนุนในรูปแบบ Dashboard",
    "iconName": "PieChart",
    "url": "https://medsup-dash.netlify.app/",
    "isActive": true,
    "type": "report"
  },
  {
    "id": "report-monthly",
    "title": "รายงานสรุปประจำเดือน",
    "description": "รายงานสรุปข้อมูลสำคัญประจำเดือน",
    "iconName": "CalendarRange",
    "url": "#",
    "isActive": false,
    "type": "report"
  },
  {
    "id": "report-stock",
    "title": "รายงานมูลค่ายาคงคลัง",
    "description": "ตรวจสอบและติดตามมูลค่ายาในคลังยา",
    "iconName": "Banknote",
    "url": "#",
    "isActive": false,
    "type": "report"
  },
  {
    "id": "report-opd",
    "title": "รายงานการใช้ยาผู้ป่วยนอก",
    "description": "สถิติและแนวโน้มการใช้ยาสำหรับผู้ป่วยนอก (OPD)",
    "iconName": "Users",
    "url": "#",
    "isActive": false,
    "type": "report"
  }
]
```

---

### Task 2: Create Resource Service (Composable)

**Action:** Create a reusable logic layer to fetch and type-check the data.

1.  Create file: `src/composables/useResources.ts`.
2.  Implement error handling and type safety.

**Implementation:**

```typescript
import { ref, onMounted } from 'vue';
import type { ResourceItem } from '@/types/resource';

export function useResources() {
  const resources = ref<ResourceItem[]>([]);
  const loading = ref(true);
  const error = ref<string | null>(null);

  async function fetchResources() {
    loading.value = true;
    error.value = null;
    
    try {
      // Since the file is in public/, we can fetch it directly.
      // We cast the response as ResourceItem[].
      const response = await fetch('/data/resources.json');
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const data: ResourceItem[] = await response.json();
      resources.value = data;
    } catch (err) {
      console.error('Failed to fetch resources:', err);
      error.value = err instanceof Error ? err.message : 'Unknown error';
    } finally {
      loading.value = false;
    }
  }

  onMounted(() => {
    fetchResources();
  });

  return {
    resources,
    loading,
    error,
    refresh: fetchResources
  };
}
```

---

### Task 3: Refactor `HomeView.vue`

**Action:** Remove static imports and replace them with the composable.

1.  Open `src/views/HomeView.vue`.
2.  Remove imports of `tools`, `reports`, `externals`, and `pharmacyResources`.
3.  Import `useResources`.
4.  Update the `<script setup>` to use reactive data.
5.  **UX Improvement:** Add a Loading skeleton or spinner.

**Implementation:**

```vue
<script setup lang="ts">
import { computed } from 'vue';
import ResourceCard from '@/components/common/ResourceCard.vue';
import { useUIStore } from '@/stores/ui';
import { useResources } from '@/composables/useResources';

// State
const store = useUIStore();
const { resources, loading, error } = useResources();

// Computed Properties for Filtering
const filteredResources = computed(() => {
  // If loading or error, return empty array
  if (loading.value || error.value) return [];

  const { currentTab, searchQuery } = store;
  
  return resources.value.filter((item) => {
    const matchesTab = currentTab === 'all' || item.type === currentTab;
    const matchesSearch = item.title
      .toLowerCase()
      .includes(searchQuery.value.toLowerCase()) || 
      item.description.toLowerCase().includes(searchQuery.value.toLowerCase());
      
    return matchesTab && matchesSearch;
  });
});
</script>

<template>
  <div class="space-y-6">
    <!-- Error State -->
    <div v-if="error" class="p-4 bg-red-100 text-red-700 rounded-xl border border-red-200">
      Error loading resources: {{ error }}
    </div>

    <!-- Loading State -->
    <div v-else-if="loading" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
      <!-- Skeleton Loader Placeholder -->
      <div v-for="n in 6" :key="n" class="bg-white h-48 rounded-2xl border border-sabot-200 animate-pulse" />
    </div>

    <!-- Data State -->
    <div v-else class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
      <ResourceCard 
        v-for="item in filteredResources" 
        :key="item.id" 
        :item="item" 
      />
      
      <div v-if="filteredResources.length === 0" class="col-span-full text-center py-10 text-sabot-400">
        ไม่พบเครื่องมือหรือรายงานที่ค้นหา
      </div>
    </div>
  </div>
</template>
```

---

### Task 4: Cleanup Deprecated Files

**Action:** Remove the old static data files to keep the codebase clean.

1.  Run the following commands in your terminal:

```bash
# Delete the old TypeScript data files
rm -rf src/data/
```

*(Note: If `src/data` contained `resource.d.ts` or important types, ensure they are moved to `src/types/` before deleting. Based on previous inspection, only TS files with data existed.)*

---

## ✅ Verification Protocol

1.  **Data Check:** Ensure `public/data/resources.json` is valid JSON (no syntax errors).
2.  **Type Check:** Run `bun run type-check`. Ensure `useResources.ts` doesn't complain about `ResourceItem` imports.
3.  **Build Check:** Run `bun run build`.
4.  **Runtime Check:**
    - Run `bun run preview`.
    - Navigate to the app.
    - **Expected:** You should see a brief "Loading" state, then the tools populate correctly.
    - **Test Search:** Try searching "warfarin". It should filter correctly.
    - **Test Tabs:** Switch between "Tools", "Reports", etc.

**Stop Condition:** If the UI shows "Error loading resources", the JSON file path is likely incorrect, or the JSON syntax is invalid.