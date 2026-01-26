# Implement Unit Tests for `PdfSplitter` Core Logic

## 1. Overview

This task is crucial for achieving a **Production-Grade** codebase. We will implement comprehensive **Unit Tests** for the `PdfSplitter` struct in `src/lib.rs`. The goal is to ensure the core business logic (PDF splitting) is correct, reliable, and resistant to future regressions. We will use the standard Rust testing framework and best practices for testing I/O-heavy code.

## 2. Technical Specifications

### 2.1. Dependency Update

Update `Cargo.toml` to include `tempfile` as a **development dependency**. This crate is essential for creating temporary directories and files, ensuring our tests are isolated and do not pollute the host file system.

```toml
[dependencies]
# ... existing dependencies (lopdf, etc.)

[dev-dependencies]
tempfile = "3.10"
```

### 2.2. Test Fixture Setup

To perform meaningful tests, a small, multi-page PDF file is required as a test fixture.

**Action:**
1.  Create a directory named `test_assets/` in the project root.
2.  Place a small, known multi-page PDF (e.g., 3-5 pages) inside this directory and name it `test_multi_page.pdf`.
3.  **Crucially**, ensure `test_assets/` is tracked by Git (or at least the test file is available to the CI/CD pipeline).

### 2.3. Unit Test Implementation (`src/lib.rs`)

Add a test module at the end of `src/lib.rs`.

```rust
// --- Add this block at the end of src/lib.rs ---

#[cfg(test)]
mod tests {
    use super::*;
    use std::fs;
    use tempfile::tempdir;

    // Define the path to the test fixture relative to the project root
    const TEST_ASSET_PATH: &str = "test_assets/test_multi_page.pdf";
    const EXPECTED_PAGE_COUNT: usize = 3; // Adjust this based on the actual test_multi_page.pdf

    /// Helper function to count the number of PDF files in a directory.
    fn count_pdf_files(dir: &Path) -> usize {
        fs::read_dir(dir)
            .unwrap()
            .filter_map(|entry| entry.ok())
            .filter(|entry| {
                entry.path().extension().map_or(false, |ext| ext == "pdf")
            })
            .count()
    }

    #[test]
    /// Test Case 1: Successful splitting of a multi-page PDF (Happy Path).
    fn test_split_pdf_success() {
        // ARRANGE: Setup temporary environment
        let splitter = PdfSplitter::new();
        let input_path = PathBuf::from(TEST_ASSET_PATH);
        
        // Create a temporary directory for the output
        let temp_output_dir = tempdir().expect("Failed to create temp dir");
        let output_dir = temp_output_dir.path();

        // ACT: Execute the core logic
        let result = splitter.split_pdf(&input_path, output_dir);

        // ASSERT: Verify the results
        assert!(result.is_ok(), "Splitting failed with error: {:?}", result.err());
        let total_pages = result.unwrap();
        
        // 1. Verify the function returns the correct page count
        assert_eq!(total_pages, EXPECTED_PAGE_COUNT, "Returned page count mismatch");

        // 2. Verify the correct number of files were created in the output directory
        let created_files = count_pdf_files(output_dir);
        assert_eq!(created_files, EXPECTED_PAGE_COUNT, "Number of output files mismatch");

        // The temporary directory and its contents will be automatically cleaned up
    }

    #[test]
    /// Test Case 2: Handling of a non-existent input file (Error Path).
    fn test_split_pdf_non_existent_input() {
        // ARRANGE: Setup
        let splitter = PdfSplitter::new();
        let non_existent_path = PathBuf::from("non_existent_file_for_test.pdf");
        let temp_output_dir = tempdir().expect("Failed to create temp dir");
        let output_dir = temp_output_dir.path();

        // ACT: Execute the core logic
        let result = splitter.split_pdf(&non_existent_path, output_dir);

        // ASSERT: Verify that an error is returned
        assert!(result.is_err(), "Splitting should fail for non-existent file");
        
        // NOTE: In the next phase (Custom Error Handling), we will assert on the specific error type.
        // For now, asserting on `is_err()` is sufficient.
    }
    
    // Future Test Case Idea: Test splitting a 1-page PDF (Edge Case)
    // Future Test Case Idea: Test splitting a corrupted PDF (Error Path)
}
```

## 3. Verification

The implementation is considered complete when the following command is executed successfully:

```bash
cargo test
```

*Expected Output*: Both `test_split_pdf_success` and `test_split_pdf_non_existent_input` must pass, demonstrating that the core `PdfSplitter` logic is functionally correct and handles basic error conditions gracefully.

## 4. Architectural Rationale

By implementing Unit Tests now, we achieve **Test-Driven Development (TDD)** principles for the core logic. The use of `tempfile` ensures that the tests are **idempotent** (running them multiple times yields the same result) and **isolated** (they do not affect the environment). This is a fundamental requirement for any world-class software project, guaranteeing that future refactoring or feature additions will not break existing functionality.
