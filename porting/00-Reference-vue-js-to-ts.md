# AGENT.md

**AI Agent Operating Contract (Phase-Aware)**

---

## MANDATORY NOTICE

This repository is **AI-assisted**.

Any AI agent (ChatGPT, Claude, Copilot, Cursor, etc.) **MUST read and follow this document completely before performing any action.**

Failure to comply with this document is considered **incorrect behavior**.

---

## 1. PROJECT CONTEXT (IMMUTABLE)

### Framework & Platform
*   **Framework:** Vue.js 3
*   **API Style:** Composition API
*   **Component Format:** Single File Components (.vue)
*   **Build Tool:** Vite
*   **State Management:** Pinia
*   **Routing:** Vue Router
*   **Runtime Environment:** Browser-only
*   **Language (current):** JavaScript
*   **Target Language:** TypeScript (strict)

### TypeScript Status
*   TypeScript is **NOT enabled yet**
*   Migration will be performed incrementally and strictly

---

## 2. PRIMARY OBJECTIVE (NON-NEGOTIABLE)

Migrate the entire codebase from JavaScript to **strict, production-ready TypeScript** while preserving **exact runtime behavior**.

### Explicitly NOT Goals
*   ❌ No refactoring for readability
*   ❌ No architectural redesign
*   ❌ No performance optimization
*   ❌ No behavior change

---

## 3. PHASE-AWARE WORKFLOW (STATE MACHINE)

### RULE
The agent **MUST operate in exactly one phase at a time**.

If the current phase is **not explicitly stated by the user**, the agent **MUST STOP and ask which phase to execute**.

### Allowed Phases (ONLY THESE)

| Phase | Name | Purpose |
| :--- | :--- | :--- |
| **Phase 0** | Sanity Check | Verify repo completeness & tooling |
| **Phase 1** | Deep Analysis | Understand architecture & types |
| **Phase 2** | Assumption Lock | Confirm inferred types & risks |
| **Phase 3** | TypeScript Migration | Write TypeScript code |
| **Phase 4** | Verification | Validate results & list risks |

---

## 4. PHASE CONTRACTS (STRICT)

### Phase 0 — Sanity Check

**Objective**
*   Ensure repository completeness and correctness before analysis.

**Allowed Actions**
*   Count files and estimate LOC.
*   Detect Vue version, tooling, libraries.
*   Identify application entry points.
*   Detect missing or truncated input.

**Forbidden**
*   ❌ Writing code
*   ❌ Inferring types
*   ❌ Making assumptions

**Hard Stop Conditions**
*   Missing files, broken imports, or truncated input.

---

### Phase 1 — Deep Analysis (NO CODE)

**Objective**
*   Fully understand architecture, data flow, and runtime behavior.

**Required Analysis**
*   Application lifecycle and entry points.
*   Component responsibilities.
*   Props / emits / slots (inferred).
*   Pinia store state, actions, getters.
*   Router params and navigation flow.
*   Shared utilities and services.
*   Implicit JavaScript behaviors.

**Strict Rules**
*   ❌ DO NOT write TypeScript
*   ❌ DO NOT refactor
*   ❌ DO NOT fix bugs

**Deliverable**
*   Structured analysis.
*   Explicit list of risks and unclear areas.

---

### Phase 2 — Assumption Lock (GATE PHASE)

**Objective**
*   Prevent incorrect migration due to hidden assumptions.

**Required Output**
*   Explicit list of:
    *   Inferred data types
    *   Ambiguous types
    *   Required assumptions
    *   Risky areas under `strict: true`

**Critical Rule**
*   The agent **MUST WAIT** for explicit user approval before proceeding to Phase 3.

**Forbidden**
*   ❌ Writing code
*   ❌ Proceeding without approval

---

### Phase 3 — TypeScript Migration (CODE PHASE)

**Objective**
*   Convert the codebase to strict, production-ready TypeScript.

#### TypeScript Configuration Assumptions
```json
{
  "strict": true,
  "noImplicitAny": true,
  "strictNullChecks": true,
  "esModuleInterop": true,
  "target": "ES2020",
  "module": "ESNext"
}
```

#### Vue-Specific Rules
*   `.vue` files MUST use:
    *   `<script setup lang="ts">` where applicable
*   Properly type:
    *   `defineProps`
    *   `defineEmits`
    *   `Ref<T>`
    *   `ComputedRef<T>`
*   Preserve `<template>` and `<style>` **EXACTLY**.
*   Ensure Volar-compatible typing.

#### Type Safety Rules
*   ❌ `any` and `unknown` are forbidden.
*   ❌ Unsafe `as` assertions are forbidden.
*   If unavoidable, MUST explain runtime guarantees in comments.

#### Code Integrity
*   Preserve runtime behavior **exactly**.
*   NO refactoring.
*   Bug fixes are allowed ONLY if required for TypeScript correctness.
    *   Format: `// FIX: explanation`

#### Output Rules
*   Output **ONLY code**.
*   Each file in its own block.
*   Required format:
    ```
    File Path: src/path/to/file.ts
    ```
    or
    ```
    File Path: src/path/to/file.vue
    ```

---

### Phase 4 — Verification & Final Report

**Objective**
*   Ensure migration correctness and transparency.

**Required Output**
*   Remaining risks.
*   Assumptions used.
*   Known limitations.
*   Areas needing human review.

**Forbidden**
*   ❌ Refactoring
*   ❌ Feature changes

---

## 5. GLOBAL TYPE RULES (APPLY TO ALL PHASES)

*   Every function MUST have:
    *   Explicit parameter types.
    *   Explicit return types.
*   Every object shape MUST be defined.
*   Implicit `any` is unacceptable.
*   Browser-only environment must be respected.
*   No Node.js APIs allowed in client code.

---

## 6. CODE STYLE & LINTER COMPLIANCE (STRICT)

The generated code **MUST** adhere to the following linting rules derived from project configuration:

### Type Definitions
*   **MUST** use `type` aliases exclusively.
*   ❌ **DO NOT** use `interface`.
    *   *Reference: `ts/consistent-type-definitions: ['error', 'type']`*

### Environment Variables
*   ❌ **FORBIDDEN:** `process.env` (Node.js legacy).
*   ✅ **MUST USE:** `import.meta.env` (Vite standard).
    *   *Reference: `node/no-process-env: ['error']`*

### Import & File Structure
*   **Imports:** Must be sorted and organized (Perfectionist rule).
*   **Filenames:** Must match `kebab-case` or `PascalCase`.
*   **Top-Level Await:** Allowed (do not workaround).

### General
*   **No Redeclare:** Check is turned `off` (allow overloads if necessary), but prefer unique naming.
*   **Console:** Avoid `console.log` in final output (`warn` level).

---

## 7. IMPORTS & DEPENDENCIES

*   Convert `require` → `import`.
*   Assume correct `@types/*` packages exist.
*   DO NOT introduce new dependencies unless explicitly instructed.

---

## 8. HARD STOP CONDITIONS (GLOBAL)

The agent **MUST STOP IMMEDIATELY** if:
1.  A file or module is missing.
2.  Input appears truncated.
3.  Type intent cannot be inferred safely.
4.  Instructions conflict with this document.

---

## 9. AUTHORITY & CONFLICT RESOLUTION

If user instructions conflict with this document:
*   **This document takes precedence.**
*   Unless the user explicitly states: > “Override AGENT.md”

---

## 10. FINAL SUCCESS CRITERIA

The migration is considered successful ONLY if:

```bash
tsc --noEmit
```

Runs with **ZERO errors** and application runtime behavior is unchanged.

---
**✅ END OF AGENT CONTRACT**
```
