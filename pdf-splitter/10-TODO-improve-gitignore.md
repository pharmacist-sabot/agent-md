# AGENT TASK: Create Comprehensive `.gitignore` File

## 1. Overview

This task involves creating a robust `.gitignore` file to ensure that only essential source code and configuration files are tracked by Git. This is a fundamental practice in professional software development, preventing the repository from being cluttered with build artifacts, temporary files, and local development settings.

## 2. Technical Specifications

The developer must create a new file named `.gitignore` in the project root directory with the following content, which is based on the standard Rust `.gitignore` template.

### 2.1. `.gitignore` Content

```gitignore
# -----------------------------------------------------------------------------
# Rust-specific files
# -----------------------------------------------------------------------------

# Build artifacts
/target/

# Cargo files
Cargo.lock
# If you want to commit Cargo.lock, comment out the line above.
# For application crates, it is generally recommended to commit Cargo.lock.

# IDE and Editor files
# Visual Studio Code
.vscode/
# IntelliJ / CLion
.idea/
# Vim
*.swp
*~

# -----------------------------------------------------------------------------
# Project-specific files (Based on previous tasks)
# -----------------------------------------------------------------------------

# Temporary directory used for local testing (from Unit Test task)
/test_assets/

# Default output directory for split pages
/output_pages/

# Old hardcoded input directory (from initial structure)
/input/

# Temporary files created by the 'tempfile' crate during testing
/tmp/

# -----------------------------------------------------------------------------
# Operating System and Miscellaneous
# -----------------------------------------------------------------------------

# macOS
.DS_Store

# Windows
Thumbs.db
ehthumbs.db

# Logs and databases
*.log
*.sqlite
*.db

# Binary files
*.exe
*.dll
*.so
*.dylib
```

### 2.2. Verification

The implementation is considered complete when the following conditions are met:

1.  **File Creation**: The file `.gitignore` is created in the project root.
2.  **Tracking Check**: Running `git status` should not show `/target/`, `/output_pages/`, or `/input/` as untracked files, confirming they are correctly ignored.

## 3. Architectural Rationale

A clean repository is a sign of a professional development team. By ignoring build artifacts and temporary files, we ensure:

*   **Smaller Repository Size**: Faster cloning and reduced storage overhead.
*   **Clean History**: The Git history remains focused on source code changes, not build noise.
*   **Reproducibility**: Developers are forced to run `cargo build` locally, ensuring they are working with fresh, reproducible build artifacts.

This task finalizes the repository setup, bringing it to a world-class standard.
