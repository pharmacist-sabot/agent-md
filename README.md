# 🤖 AGENT.md Protocol: Central Command

> **The Operating System for Autonomous AI Agents.**
> A collection of immutable contracts, architectural constraints, and phase-aware protocols for software development.

![Status](https://img.shields.io/badge/Status-Active-success)
![Agents](https://img.shields.io/badge/Total_Projects-8-blue)
![Standard](https://img.shields.io/badge/Protocol-v2.0-orange)

---

## 🧭 Navigation & Project Registry

| Project / Domain | Tech Stack | Status Overview | Context |
| :--- | :--- | :--- | :--- |
| **[🌿 herbs-app](./herbs-app/)** | Vue 3, Pinia, Google Sheets API | 🟡 In Progress | Performance optimization & E2E Testing |
| **[💊 high-alert-drugs-app](./high-alert-drugs-app/)** | Vue 3, Supabase | 🔴 High Priority | Migration to TypeScript & Pinia |
| **[🩸 warfarin-app](./warfarin-app/)** | Rust (WASM), Vue 3 | 🟢 Stable | Logic Safety, PWA & Unit Tests |
| **[📊 medsup-dash](./medsup-dash/)** | Vue 3, Tailwind, Chart.js | 🟡 In Progress | Dashboard Logic & Error Handling |
| **[🏥 pharmacy-hub-sabot](./pharmacy-hub-sabot/)** | Vue 3, Firebase Hosting | 🟢 Stable | PWA, Security & Data Layer |
| **[🦀 rust-roadmap](./rust-roadmap/)** | Rust, Leptos, SVG | 🛠️ Construction | Architectural backbone & Content |
| **[📝 rxdevnotes](./rxdevnotes/)** | Astro, TypeScript | 🟡 In Progress | Content Structure & Features |
| **[🛠️ Porting & Migration](./porting/)** | Cross-Framework | 📚 Library | Reference contracts for migration |

---

## 🚦 Naming Convention & Workflow

Files in this repository follow a strict naming convention to denote **Order of Execution** and **Current Status**.

**Format:** `[Sequence]-[Status]-[Slug].md`

### 1. Sequence (00-99)
Indicates the logical dependency. **Agents must process lower numbers first.**
*   `01`: Foundation / Infrastructure / Types
*   `05`: Logic / Features
*   `09`: Testing / CI-CD

### 2. Status Tags
| Tag | Meaning | Action Required |
| :--- | :--- | :--- |
| **`TODO`** | Pending | Ready to be executed by an AI Agent. |
| **`WIP`** | Work In Progress | Currently being implemented; may contain partial context. |
| **`DONE`** | Completed | Archived context. Use for reference/context only. |
| **`SKIP`** | Deprecated | Do not process. |

### 3. Usage (The Bootstrap Prompt)
To activate an agent for a specific task, copy the content of the target `.md` file and prepend this instruction:

> **SYSTEM PROMPT:**
> "You are an autonomous developer agent. Read the attached `AGENT.md` file. This file is a **BINDING CONTRACT**. You must strictly follow the constraints, stack, and phase-aware workflow defined within. Do not deviate. Confirm your understanding of the **Primary Objective** before generating code."

---

## 🏗️ The Protocol Standard (Anatomy of an Agent)

Every `AGENT.md` file in this repository strictly adheres to the following structure:

1.  **Context & Role:** Defines *who* the AI is (e.g., "Senior Rust Architect").
2.  **Immutable Constraints:** Facts that cannot be changed (e.g., "Use Vue 3 Composition API").
3.  **Phase-Aware Workflow:** Steps that must be done in order (Analysis -> Approval -> Code).
4.  **Negative Constraints:** What is explicitly *forbidden* (e.g., "No jQuery", "No `any` type").
5.  **Quality Gates:** Tests/Linters that must pass for the task to be considered "Done".

---

## 🤝 Contributing & Extension

To add a new project:
1.  Create a folder named after the project `kebab-case`.
2.  Add a `README.md` inside that folder describing the project stack.
3.  Create agent files starting with `01-TODO-...`.

To add a new Task to an existing project:
1.  Check the highest existing sequence number.
2.  Create `[Next_Number]-TODO-[task-name].md`.

---

<div align="center">
  <i>"Control the Agent, Control the Code."</i>
</div>