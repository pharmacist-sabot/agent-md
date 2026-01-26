# Implement Secure Path Handling (Path Traversal Prevention)

## 1. Overview

This task addresses a critical security concern: **Path Traversal** (or Directory Traversal). When constructing file paths based on user input (like the output directory), improper concatenation can allow an attacker to write files to arbitrary locations using relative paths like `../..`. We will refactor the path construction logic in `src/lib.rs` to use the secure, idiomatic Rust method: `PathBuf::join`.

## 2. Technical Specifications

### 2.1. Update Custom Error Type (`src/lib.rs`)

We need to add a specific error variant to `SplitterError` to handle potential security violations or invalid path constructions.

**Required Code Structure in `src/lib.rs` (within `SplitterError` enum):**

```rust
#[derive(Debug, Error)]
pub enum SplitterError {
    // ... existing variants (FileNotFound, Io, Pdf) ...

    /// Error variant for when the constructed output path is invalid or unsafe.
    #[error("Output path construction failed: {0}")]
    OutputPathError(String), 
    
    // ... existing variant (InvalidPathFormat) ...
}
```

### 2.2. Refactor Path Construction (`src/lib.rs`)

The path construction logic inside `PdfSplitter::split_pdf` must be replaced.

**Goal:** Replace the string-based path concatenation:
```rust
// Inefficient and potentially unsafe string concatenation:
let output_filename = format!(
    "{}/page_{:03}.pdf",
    output_dir.to_str().unwrap(), // Unsafe unwrap, and string concatenation
    page_number
);
new_doc.save(&output_filename)?;
```

**Action:** Use `PathBuf::join` for secure and platform-agnostic path construction.

**Required Code Structure in `src/lib.rs` (inside `PdfSplitter::split_pdf` loop):**

```rust
// ... inside the loop for page_number ...

// 1. Construct the filename component securely
let filename = format!("page_{:03}.pdf", page_number);

// 2. Use PathBuf::join for secure, platform-agnostic path concatenation
// This method correctly handles path separators (e.g., / on Linux, \ on Windows)
let output_path = output_dir.join(filename);

// 3. Save the document using the securely constructed path
new_doc.save(&output_path)?; 

// ... rest of the loop ...
```

### 2.3. Remove Unsafe Path Conversion

The previous string-based approach required converting `output_dir` to a string using `.to_str().unwrap()`. The new approach using `PathBuf::join` works directly with `Path` and `PathBuf` types, eliminating the need for this potentially unsafe conversion and the `InvalidPathFormat` error variant for this specific use case.

**Action:**
1.  Remove the logic that converts `output_dir` to a string and handles `InvalidPathFormat` for the output path construction.
2.  (Optional but Recommended) If the `InvalidPathFormat` variant is no longer used anywhere else, it should be removed from `SplitterError` to keep the error surface clean.

## 3. Verification

The implementation is considered complete when the following conditions are met:

1.  **Successful Compilation**: `cargo build` completes without warnings or errors.
2.  **Test Pass**: `cargo test` still passes. The Unit Tests must be updated to reflect the new path construction logic, but the test's outcome (files created) should remain the same.
3.  **Security Check (Conceptual)**: The code no longer uses string formatting (`format!`) or manual path separators (`/`) to combine the directory and the filename. It relies entirely on `PathBuf::join`, which is inherently resistant to simple Path Traversal attacks.

## 4. Architectural Rationale

Using `PathBuf::join` is the **canonical best practice** in Rust for path manipulation. It ensures:

*   **Platform Agnosticism**: Correctly handles path separators on all operating systems (Linux, Windows, macOS).
*   **Security**: It treats the second argument as a filename or relative path component, preventing it from overriding the base path with absolute or malicious relative paths (like `../../`).

This refactoring solidifies the application's security posture against a common class of vulnerabilities.
