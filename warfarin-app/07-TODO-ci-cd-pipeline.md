# Task: Setup CI/CD Pipeline with GitHub Actions

## Objective
Automate the Quality Assurance process using GitHub Actions. Every Pull Request (PR) must pass a strict pipeline containing Rust Tests, Rust Linting (Clippy), TypeScript/Vue Linting (ESLint), and a Full Build (WASM + Frontend).

**Goals:**
1.  **Prevent Bad Code:** Fail the PR build if tests fail or linting errors exist.
2.  **Performance:** Implement aggressive caching to keep CI times under 3-5 minutes.
3.  **Standards:** Use industry-standard actions (`actions/checkout`, `actions/setup-node`, `dtolnay/rust-toolchain`).

---

## Implementation Guide

### 1. Workflow Configuration
Create the following file structure: `.github/workflows/ci.yml`

This workflow runs on **Push** and **Pull Request** events.

```yaml
name: Quality Gate

on:
  push:
    branches: ["main", "master"]
  pull_request:
    branches: ["main", "master"]

jobs:
  quality-gate:
    runs-on: ubuntu-latest

    steps:
      # 1. Checkout Code
      - name: Checkout repository
        uses: actions/checkout@v4

      # 2. Setup Environments
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "22"
          cache: "npm" # Auto-caches node_modules

      - name: Setup Rust
        uses: dtolnay/rust-toolchain@stable
        with:
          components: clippy # Ensure clippy is installed

      # 3. Caching (Crucial for speed)
      # Cache Rust dependencies to avoid re-downloading crates every run
      - name: Cache Rust Cargo Registry
        uses: actions/cache@v4
        with:
          path: ~/.cargo/registry
          key: ${{ runner.os }}-cargo-registry-${{ hashFiles('**/Cargo.lock') }}

      - name: Cache Rust Cargo Index
        uses: actions/cache@v4
        with:
          path: ~/.cargo/git
          key: ${{ runner.os }}-cargo-index-${{ hashFiles('**/Cargo.lock') }}

      # Cache Cargo Build (wasm-pack compilation artifacts)
      - name: Cache Rust Cargo Build
        uses: actions/cache@v4
        with:
          path: target
          key: ${{ runner.os }}-cargo-build-target-${{ hashFiles('**/Cargo.lock') }}

      # 4. Install Dependencies
      - name: Install Node Dependencies
        run: npm ci

      # Install wasm-pack if not cached (or ensure it is available)
      # Note: Since we can't easily cache binaries with actions/cache, we install quickly.
      - name: Install wasm-pack
        run: cargo install wasm-pack --quiet

      # 5. Rust Logic Layer Checks
      - name: Run Rust Tests
        working-directory: ./warfarin_logic
        run: cargo test --verbose

      - name: Run Rust Clippy (Strict Mode)
        working-directory: ./warfarin_logic
        # -D warnings forces all clippy warnings to be treated as errors
        run: cargo clippy --all-targets -- -D warnings

      # 6. Frontend Layer Checks
      - name: Run ESLint
        run: npm run lint

      # 7. Full Build Integration Test
      # This verifies that WASM builds correctly AND Vue compiles with WASM dependencies
      - name: Build Project
        run: npm run build
        env:
          # Suppress progress bars to keep logs clean
          CARGO_TERM_COLOR: always
```

---

## Phase 2: Strict Enforcement Settings

### Rust (`warfarin_logic/src/lib.rs`)
Ensure `Clippy` passes cleanly. The pipeline enforces `-D warnings`, meaning common code smells (like unused variables or incorrect clone usage) will block the merge.

### TypeScript / Vue (`.eslintrc` or `eslint.config.js`)
The pipeline runs `npm run lint`. Ensure your `package.json` script is configured to fail on errors (default behavior of ESLint).

```json
// package.json
"scripts": {
  "lint": "eslint . --max-warnings 0", // This ensures the CI fails if there are ANY errors/warnings
  "test": "cargo test"
}
```

---

## Validation Steps

1.  **Push a dummy commit** to a new branch.
2.  **Open a Pull Request.**
3.  Observe the **"Checks"** tab in GitHub.
4.  **Green Tick:**
    *   `Run Rust Tests`: Must pass (~30s - 1min).
    *   `Run Rust Clippy`: Must pass (~10s).
    *   `Run ESLint`: Must pass (~10s).
    *   `Build Project`: Must pass (~1min - 2min).
5.  **Branch Protection Rules:**
    *   (Manual Step for User): Go to Repository Settings -> Branches -> Add Rule.
    *   Check "Require status checks to pass before merging" and select "Quality Gate".
    *   This effectively locks the `main` branch from bad code.

## Notes for AI Agent
*   **Path Context:** Ensure the workflow checks the correct working directory (`./warfarin_logic`) for Rust commands.
*   **wasm-pack:** The workflow installs `wasm-pack` explicitly because the GitHub Action runner image does not include it by default.
*   **Permissions:** No extra `permissions` block is needed for running tests/lint/build, as we are only reading and building code, not deploying to external environments in this workflow.