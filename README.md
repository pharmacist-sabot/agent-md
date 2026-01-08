<div align="center">

  # AGENT.md Protocol

**The Operating System for Autonomous AI Agents.**

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Status](https://img.shields.io/badge/status-active-success.svg)
![Standard](https://img.shields.io/badge/standard-agent--md-orange.svg)

---
</div>

## 🧠 Abstract

As LLMs evolve from "chat bots" to "agentic workers," the primary bottleneck becomes **control**. Simple prompts result in non-deterministic behavior, architectural drift, and "hallucinated" best practices.

This repository defines **`AGENT.md`**: a standardized, contract-driven protocol that transforms AI agents (ChatGPT, Claude, Cursor, Copilot) into **deterministic, phase-aware, and constraint-bound** workers.

An `AGENT.md` file is not documentation; it is a **binding contract** between the human architect and the machine executor.

---

## 🏛️ Core Principles

### 1. Immutable Context
Defines "Source of Truth" facts (tech stack, environment variables, framework versions) that the AI is forbidden from hallucinating or overriding.

### 2. Phase-Aware State Machines
Agents are forced to operate in explicit states (e.g., `Analysis` → `Approval` → `Execution`). This prevents "jumping the gun" and writing code before understanding the system.

### 3. Negative Constraints
Explicitly defines what is **NOT** allowed (e.g., "No refactoring during migration," "No `any` types," "No CDN links"). This is often more important than defining what *is* allowed.

### 4. Quality Gates
Hard enforcement of tooling standards (Clippy, Rustfmt, ESLint, Prettier) directly within the prompt, ensuring the code passes CI/CD before it is even written.

---

## 📚 The Contract Library

This repository contains production-ready contracts for various scenarios. Copy the relevant `.md` file to your project root to activate the agent persona.

### 🛠️ Porting & Migration
**Location:** `porting/`

Strict contracts for translating codebases between languages or frameworks with zero behavioral changes.

| Contract | Target | Key Constraints |
| :--- | :--- | :--- |
| **[vue-js-to-ts.md](porting/vue-js-to-ts.md)** | Vue 3 (JS → TS) | Phase-locked execution. `strict: true` enforced. Zero refactoring allowed. |

### ⚙️ Systems & Rust
**Location:** `rust-roadmap/`

Contracts designed for high-performance, type-safe environments where correctness and compile-time guarantees are paramount.

| Contract | Domain | Focus |
| :--- | :--- | :--- |
| **[backbone-project.md](rust-roadmap/backbone-project.md)** | Data Modeling | Strict data structure definitions, canonical sequencing, file-system organization. |
| **[rust-css.md](rust-roadmap/rust-css.md)** | Styling | "Rust-first" CSS. No CDNs. Deterministic builds via Trunk/Lightning CSS. |
| **[rust-roadmap.md](rust-roadmap/rust-roadmap.md)** | Core Architecture | Leptos/SVG focus. Clippy/Rustfmt as hard gates. Explicit > Implicit. |

### 🌐 Web Application (Astro)
**Location:** `rxdevnotes/`

Granular contracts for implementing specific features or maintaining content-heavy sites.

| Contract | Task | Strategy |
| :--- | :--- | :--- |
| **[refactor-blog-contents-structure.md](rxdevnotes/refactor-blog-contents-structure.md)** | Refactoring | Directory-based (Page Bundle) migration. Safe history preservation (`git mv`). |
| **[related-post.md](rxdevnotes/related-post.md)** | Feature Implementation | Algorithm-based scoring for content recommendations. Reuse over duplication. |
| **[social-share-component.md](rxdevnotes/social-share-component.md)** | Feature Implementation | Native API usage over 3rd party libs. Accessibility focused. |

---

## 🚀 Quickstart

### 1. Installation
Identify the contract that matches your project's needs and copy it to your root directory.

```bash
# Example: Migrating a Vue project to TypeScript
cp porting/vue-js-to-ts.md ./AGENT.md
```

### 2. Activation
When initializing an AI session (Composer, ChatGPT, Claude Projects), provide the context immediately:

> **System Prompt:**
> "Read the `AGENT.md` file in the root directory. Adopt the persona, constraints, and workflow defined therein. Do not proceed until you acknowledge your current phase."

### 3. Enforcement (The Hard Stop)
If the agent drifts—skipping analysis or violating the contract—redirect it instantly:

> **Correction:**
> "You are violating Section [X] of AGENT.md. Revert to Phase [Y]. Await further instructions."

---

## 🏗️ Anatomy of a Contract

A high-quality `AGENT.md` follows this structure:

1.  **MANDATORY NOTICE:** Explicit warning that compliance is required for correctness.
2.  **PROJECT CONTEXT:** Immutable facts (Framework, Build Tool, Runtime).
3.  **PRIMARY OBJECTIVE:** The single non-negotiable goal.
4.  **PHASE-AWARE WORKFLOW:** A state machine table (e.g., Phase 0: Sanity, Phase 1: Analysis).
5.  **GLOBAL RULES:** Linting rules, typing strictness, and forbidden patterns.
6.  **HARD STOP CONDITIONS:** Criteria for immediate termination (e.g., missing dependencies).

---

## 🤝 Contributing

We accept contracts for any stack. The standard for submission is high. Contracts must be:

*   **Deterministic:** No "use your best judgment" clauses. Be explicit.
*   **Testable:** Rules should be verifiable by linters or compilers.
*   **Scenario-Specific:** Generic prompts are rejected; specific operational contracts are accepted.

To contribute:
1.  Fork the repository.
2.  Create your contract in `examples/` or the relevant directory.
3.  Ensure it strictly follows the **Anatomy** defined above.
4.  Submit a PR.

---

## 📜 License

Distributed under the MIT License.

---

<p align="center">
  <i>"Control the Agent, Control the Code."</i>
</p>
