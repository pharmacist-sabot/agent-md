# AGENT.md Protocol

**The Operating System for AI Coding Agents.**

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Status](https://img.shields.io/badge/status-active-success.svg)

This repository serves as a collection of **AI Operating Contracts**—standardized Markdown files designed to enforce strict behavior, phase-aware workflows, and non-negotiable constraints on AI agents (ChatGPT, Claude, Cursor, Copilot, etc.) within software projects.

## ❓ What is `AGENT.md`?

As AI agents become more autonomous, simple prompts are no longer sufficient for complex tasks like refactoring, migrations, or architectural changes.

**`AGENT.md`** is a context file placed in the root of your project. It acts as a **binding contract** between the developer and the AI. It transforms the AI from a chaotic suggestion engine into a **deterministically behaving worker**.

### Core Concepts

1.  **Immutable Context:** Defines the tech stack and environment facts that the AI must never hallucinate.
2.  **Phase-Aware Workflow:** Forces the AI to operate as a *State Machine* (e.g., Analysis → Approval → Execution).
3.  **Negative Constraints:** Explicitly lists what is **NOT** allowed (e.g., "No refactoring for readability").
4.  **Linter Compliance:** Hard-coded style rules (e.g., `type` vs `interface`, `import.meta.env` vs `process.env`).

---

## 🚀 Usage

### 1. Installation
Copy the relevant `AGENT.md` file from this repository into the **root** of your project.

### 2. Activation
When starting a session with an AI (Cursor Composer, ChatGPT, Claude Project), initialize it with:

> "Read AGENT.md. Adopt the persona and constraints defined therein. Do not proceed until you acknowledge your current phase."

### 3. Enforcement
The `AGENT.md` file includes a **"Hard Stop"** mechanism. If the AI deviates from the contract (e.g., skipping the "Analysis Phase" and jumping straight to coding), point it back to the document:

> "You are violating Section 3 of AGENT.md. Revert to Phase 1."

---

## 🤝 Contributing

We welcome contributions! If you have developed an `AGENT.md` for a specific stack (React, Python, Go, Legacy Refactoring, etc.), please submit a Pull Request.

1.  Fork the repository.
2.  Create a folder under `examples/` or `templates/`.
3.  Add your `AGENT.md`.
4.  Submit a PR with a description of the use case.

---

## 📜 License

Distributed under the MIT License. See `LICENSE` for more information.

---

<p align="center">
  <i>"Control the Agent, Control the Code."</i>
</p>
