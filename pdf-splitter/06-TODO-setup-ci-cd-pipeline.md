# Enforce Coding Standards via CI/CD Pipeline

## 1. Overview

This task implements the final layer of quality assurance by integrating automated checks for coding standards into the project's Continuous Integration (CI) pipeline. We will use **GitHub Actions** to enforce two essential Rust tools:

1.  **`rustfmt`**: Ensures consistent code formatting across the entire codebase.
2.  **`clippy`**: A powerful linter that catches common mistakes, potential bugs, and stylistic issues, pushing the code quality towards idiomatic Rust.

The goal is to make the build fail if any code is not properly formatted or contains linter warnings, thus guaranteeing a high and consistent code standard.

## 2. Technical Specifications

### 2.1. CI/CD Pipeline Setup

Create the GitHub Actions workflow file at `.github/workflows/ci.yml`.

**Required Code Structure in `.github/workflows/ci.yml`:**

```yaml
name: Rust CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

env:
  CARGO_TERM_COLOR: always

jobs:
  build_and_test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Install Rust toolchain
      uses: dtolnay/rust-toolchain@stable
      with:
        toolchain: stable
        components: rustfmt, clippy # Ensure both tools are installed

    # --- 1. Enforce Code Formatting (rustfmt) ---
    - name: Check code formatting
      run: cargo fmt --check
      # The --check flag ensures the command fails if any file is not formatted.
      # This forces developers to run 'cargo fmt' locally before committing.

    # --- 2. Run Linter (clippy) ---
    - name: Run clippy linter
      run: cargo clippy -- -D warnings
      # The '-D warnings' flag treats all clippy warnings as errors.
      # This is a world-class standard for enforcing high code quality.

    # --- 3. Run Unit Tests ---
    - name: Run tests
      run: cargo test --verbose

    # --- 4. Build Release Binary (Final Check) ---
    - name: Build release
      run: cargo build --release
```

### 2.2. Local Configuration (Optional but Recommended)

To ensure developers can easily pass the CI checks locally, it is best practice to include a `rust-toolchain.toml` file.

**Required Code Structure in `rust-toolchain.toml` (in project root):**

```toml
[toolchain]
channel = "stable"
components = ["rustfmt", "clippy"]
```

### 2.3. Verification

The implementation is considered complete when the following conditions are met:

1.  **File Creation**: The file `.github/workflows/ci.yml` is created with the specified content.
2.  **Successful CI Run**: A Pull Request or Push to the `main` branch triggers the GitHub Action, and all four steps (`Check code formatting`, `Run clippy linter`, `Run tests`, `Build release`) pass successfully.
3.  **Failure Test**: Introduce a deliberate formatting error (e.g., inconsistent indentation) and confirm that the `Check code formatting` step fails, demonstrating the enforcement mechanism is active.

## 3. Architectural Rationale

Integrating `rustfmt` and `clippy` into the CI pipeline transforms coding standards from a guideline into an **enforced rule**.

| Tool | Purpose | Standard Enforced | Impact on Quality |
| :--- | :--- | :--- | :--- |
| **`rustfmt --check`** | Consistency | Ensures all code adheres to a single, agreed-upon style. | Eliminates style debates, improves readability, and reduces cognitive load. |
| **`clippy -D warnings`** | Quality & Idiomacy | Treats all linter suggestions as critical errors. | Catches subtle bugs, enforces idiomatic Rust patterns, and prevents technical debt accumulation. |

This setup is a non-negotiable requirement for any project aiming for a production-grade, maintainable codebase.
