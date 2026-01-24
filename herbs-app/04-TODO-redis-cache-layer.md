# Redis Cache Layer Implementation (Vercel KV)

**Role:** Backend Performance Engineer
**Objective:** Mitigate slow Google Sheets API latency by introducing a Vercel KV (Redis) Cache layer using a Serverless Function Proxy.
**Architectural Shift:** The Frontend will no longer call Google Apps Script directly. It will call a `/api/herbs` endpoint on Vercel, which checks the Cache first.
**Target Deployment:** Vercel (Required for Vercel KV).

---

## Phase 1: Infrastructure Setup (Vercel Console)

1.  **Create KV Database:**
    *   Go to your Vercel Dashboard -> Project -> Storage.
    *   Create a new database -> Select **KV** (Redis compatible).
    *   Copy the Environment Variables provided:
        *   `KV_URL`
        *   `KV_REST_API_URL`
        *   `KV_REST_API_TOKEN`

2.  **Local Environment Setup:**
    *   Add these variables to your local `.env` file:
        ```env
        # Vercel KV Credentials
        KV_REST_API_URL=redis://...
        KV_REST_API_TOKEN=...

        # Original Google Sheet URL (Keep this for the backend function)
        VITE_GOOGLE_API_URL=https://script.google.com/macros/s/YOUR_ID/exec
        ```

---

## Phase 2: Install Dependencies

1.  **Install Vercel SDK:**
    We need the official Vercel KV client.
    ```bash
    bun add @vercel/kv
    ```

---

## Phase 3: Create Serverless Proxy (The Cache Logic)

Since this is a Vue/Vite project, we need to create an API route that Vercel recognizes.

1.  **Create API Directory:**
    Create `api/` in the root directory.

2.  **Create Proxy Function:**
    *   **File:** `api/herbs.ts`
    *   **Logic:**
        1.  Check if data exists in Redis.
        2.  If YES -> Return cached data immediately (Fast).
        3.  If NO -> Fetch from Google Apps Script.
        4.  Store in Redis with a TTL (e.g., 15 minutes).
        5.  Return data.
        6.  **Fallback:** If Google Script fails, try to return stale data if possible (Graceful Degradation).

    **Code Template:**
    ```typescript
    // api/herbs.ts
    import { NextRequest, NextResponse } from 'next/server'; // Use standard Web API types if not using Next.js, but Vercel uses this signature
    import { kv } from '@vercel/kv';

    // Use standard Vercel Runtime
    export const config = {
      runtime: 'nodejs',
    };

    export default async function handler(req: NextRequest) {
      const CACHE_KEY = 'herbs_data';
      const TTL_SECONDS = 900; // 15 minutes

      // 1. CORS Headers (Allow frontend to access this API)
      const headers = {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'GET, OPTIONS',
        'Content-Type': 'application/json',
      };

      if (req.method === 'OPTIONS') {
        return new NextResponse(null, { status: 200, headers });
      }

      try {
        // 2. Check Cache
        let data = await kv.get<any>(CACHE_KEY);

        if (data) {
          console.log('[Cache] HIT');
          return NextResponse.json(data, { headers });
        }

        console.log('[Cache] MISS');

        // 3. Fetch from Google Apps Script
        const googleApiUrl = process.env.VITE_GOOGLE_API_URL;
        if (!googleApiUrl) {
          throw new Error('Missing VITE_GOOGLE_API_URL');
        }

        const res = await fetch(googleApiUrl);
        
        if (!res.ok) {
          throw new Error(`Google API failed: ${res.status}`);
        }

        data = await res.json();

        // 4. Store in Cache
        await kv.set(CACHE_KEY, data, { ex: TTL_SECONDS });

        return NextResponse.json(data, { headers });

      } catch (error: any) {
        console.error('[API Error]', error);
        
        // 5. Graceful Degradation: Try to return Stale Cache if available
        // Note: kv.get only gets valid keys. You might need a separate key for stale data if you want this advanced logic.
        // For now, return error.
        
        return NextResponse.json(
          { error: 'Failed to fetch herbs', message: error.message },
          { status: 500, headers }
        );
      }
    }
    ```

    *Note: If you are NOT using Next.js, you might need to structure this as `api/herbs/index.ts` or configure `vercel.json`. However, for Vite projects on Vercel, a simple `api/herbs.ts` often works if configured correctly, or you can use the Express/Hono approach. The above uses the Next.js request signature which is often compatible or can be easily adapted to a `req, res` handler if using Vercel's Node adapter.*

    **Alternative for Pure Vite on Vercel (Express/Hono approach is safer):**
    Since Vite + Vercel can be tricky with `api/` folders without Next.js, here is a safer approach using a standard Node.js function signature if the above fails:
    
    ```typescript
    // api/herbs.ts (Standard Node.js export)
    import { kv } from '@vercel/kv';

    export default async (req: any, res: any) => {
      // Enable CORS
      res.setHeader('Access-Control-Allow-Credentials', true);
      res.setHeader('Access-Control-Allow-Origin', '*');
      if (req.method === 'OPTIONS') {
        return res.status(200).end();
      }

      const CACHE_KEY = 'herbs_data';

      try {
        // Check Cache
        let data = await kv.get(CACHE_KEY);
        if (data) return res.status(200).json(data);

        // Fetch Google Sheet
        const response = await fetch(process.env.VITE_GOOGLE_API_URL);
        const jsonData = await response.json();

        // Set Cache
        await kv.set(CACHE_KEY, jsonData, { ex: 600 }); // 10 mins

        return res.status(200).json(jsonData);
      } catch (error) {
        return res.status(500).json({ error: 'Internal Server Error' });
      }
    };
    ```

---

## Phase 4: Frontend Migration

Update the frontend to call your new Vercel Proxy instead of the Google URL directly.

1.  **Update Service:**
    *   **File:** `src/services/herbs-service.ts`
    *   **Action:** Change the `getHerbs` endpoint.

    **Refactored Code:**
    ```typescript
    // src/services/herbs-service.ts
    import type { Herb } from '@/types/Herb';

    const API_BASE = '/api/herbs'; // Relative path (calls the serverless function)

    export const herbsService = {
      async getHerbs(): Promise<Herb[]> {
        // In development, this proxies to Vercel Dev server if configured, 
        // or you might need to use the full Vercel URL if testing locally against production.
        // For local dev with Vite proxy, ensure vite.config.ts proxies /api to Vercel URL 
        // OR run this locally using Vercel CLI (`vercel dev`).
        
        const response = await fetch(API_BASE);

        if (!response.ok) {
          throw new Error(`Error fetching herbs: ${response.statusText}`);
        }

        const data: Herb[] = await response.json();
        return data;
      }
    };
    ```

2.  **Local Development Setup (Crucial):**
    *   Since `api/herbs.ts` is a serverless function, standard `bun run dev` won't run it unless you proxy it.
    *   **Solution:** Run the app using **Vercel CLI** locally.
    
    *   Install Vercel CLI: `bun add -D vercel`
    *   Update `package.json` scripts:
        ```json
        "dev": "vercel dev", // Instead of vite
        "dev:legacy": "vite" // Keep for UI only
        ```

---

## Phase 5: Safety & Deployment

1.  **Deploy to Vercel:**
    *   Push to GitHub.
    *   Ensure Environment Variables (`KV_REST_API_URL`, etc.) are set in Vercel Project Settings.
    *   Deploy.

2.  **Monitor Cache Hits:**
    *   Check Vercel Functions Logs to see if you see `[Cache] HIT` or `[Cache] MISS`.
    *   First load should be `MISS`. Second load (within 10 mins) should be `HIT`.

3.  **Fallback Strategy:**
    *   If the Redis server is down, the `try/catch` block in the API route will execute.
    *   Ensure your frontend `useHerbs` composable handles the error gracefully (e.g., by using the existing Toast notification from previous steps).

## Final Checklist:
- [ ] Vercel KV Database created and connected.
- [ ] `@vercel/kv` installed.
- [ ] `api/herbs.ts` created with Cache logic.
- [ ] `src/services/herbs-service.ts` updated to point to `/api/herbs`.
- [ ] App run locally via `vercel dev`.
- [ ] Logs show `[Cache] MISS` followed by `[Cache] HIT`.

**End of Instructions.**