# Implement Custom Error Type with `thiserror`

## 1. Overview

This task elevates the application's reliability by implementing a **Custom Error Type** named `SplitterError`. This is a critical step in achieving world-class error handling, allowing the application to clearly distinguish between different failure modes (e.g., File Not Found, PDF Parsing Error, I/O Error) and provide meaningful feedback to the user or calling program. We will use the popular and idiomatic Rust crate, **`thiserror`**, for declarative error definition.

## 2. Technical Specifications

### 2.1. Dependency Update

Update `Cargo.toml` to include `thiserror` as a dependency.

```toml
[dependencies]
lopdf = "0.39.0"
clap = { version = "4.5", features = ["derive"] }
thiserror = "1.0" # New dependency
```

### 2.2. Define Custom Error Type (`src/lib.rs`)

Define the `SplitterError` enum in `src/lib.rs`. This enum will encapsulate all possible errors that can occur within the `PdfSplitter` service.

**Required Code Structure in `src/lib.rs`:**

```rust
// --- Add this block near the top of src/lib.rs ---
use thiserror::Error;

/// Custom Error Type for the PdfSplitter service.
#[derive(Debug, Error)]
pub enum SplitterError {
    /// Error variant for when the input PDF file is not found.
    #[error("Input file not found: {path}")]
    FileNotFound { path: std::path::PathBuf },

    /// Error variant for I/O operations (e.g., creating directory, saving file).
    #[error("I/O error: {0}")]
    Io(#[from] std::io::Error),

    /// Error variant for issues during PDF parsing or manipulation by lopdf.
    #[error("PDF processing error: {0}")]
    Pdf(#[from] lopdf::Error),

    /// Error variant for when a path cannot be converted to a string (e.g., non-UTF8 paths).
    #[error("Invalid path format: Path could not be converted to a valid string.")]
    InvalidPathFormat,
}

// Update the PdfSplitter struct's public method signature
// to return the new custom error type.
impl PdfSplitter {
    // ...
    pub fn split_pdf(
        &self,
        input_path: &std::path::Path,
        output_dir: &std::path::Path,
    ) -> Result<usize, SplitterError> { // <--- CHANGE: Use SplitterError
        // ... implementation
    }
}
```

### 2.3. Refactoring Error Propagation (`src/lib.rs`)

Modify the `PdfSplitter::split_pdf` function to use the new `SplitterError` type. The `#[from]` attributes on `Io` and `Pdf` variants allow us to use the `?` operator seamlessly for `std::io::Error` and `lopdf::Error`.

**Key Changes in `src/lib.rs`:**

1.  **Change Function Signature**:
    ```rust
    // BEFORE:
    // pub fn split_pdf(...) -> lopdf::Result<usize> {
    // AFTER:
    pub fn split_pdf(...) -> Result<usize, SplitterError> {
    ```

2.  **Handle File Not Found (Custom Error)**: The initial check for file existence must now return the custom error.
    ```rust
    // BEFORE:
    // if !input_path.exists() { /* ... panic or exit ... */ }
    
    // AFTER:
    if !input_path.exists() {
        return Err(SplitterError::FileNotFound { path: input_path.to_path_buf() });
    }
    ```

3.  **Handle Path Conversion (Custom Error)**: Replace the unsafe `.unwrap()` with a proper error conversion.
    ```rust
    // BEFORE:
    // output_dir.to_str().unwrap()
    
    // AFTER:
    let output_dir_str = output_dir.to_str()
        .ok_or(SplitterError::InvalidPathFormat)?; // Use ? with custom error
        
    let output_filename = format!(
        "{}/page_{:03}.pdf",
        output_dir_str,
        page_number
    );
    ```

4.  **Propagate Errors**: All existing uses of the `?` operator for `fs::create_dir_all`, `Document::load`, and `new_doc.save` will now automatically convert to `SplitterError` due to the `#[from]` attributes.

### 2.4. Update CLI Entry Point (`src/main.rs`)

The `src/main.rs` file must be updated to handle the new `SplitterError` returned by the library.

**Key Changes in `src/main.rs`:**

1.  **Change Function Signature**:
    ```rust
    // BEFORE:
    // fn main() -> lopdf::Result<()> {
    // AFTER:
    fn main() -> Result<(), Box<dyn std::error::Error>> { // Use a generic error type for main
    ```

2.  **Handle Error Display**: The error returned by `splitter.split_pdf(...)` must be handled gracefully.
    ```rust
    // ...
    match splitter.split_pdf(input_path, output_dir) {
        Ok(total_pages) => {
            // Success message
            println!(
                "\n✅ Successfully split {} pages into '{}'",
                total_pages,
                output_dir.display()
            );
            Ok(())
        }
        Err(e) => {
            // Display the user-friendly error message
            eprintln!("Error: {}", e);
            // Exit with a non-zero status code
            std::process::exit(1);
        }
    }
    ```
    *Note: For simplicity in `main`, we can also use `splitter.split_pdf(...)?;` and change `main`'s return type to `Result<(), SplitterError>` if we implement `From<SplitterError> for Box<dyn Error>`. However, the `match` block provides better control over the exit code and user-facing message.*

## 3. Verification

The implementation is considered complete when the following conditions are met:

1.  **Successful Compilation**: `cargo build` completes without warnings or errors.
2.  **Test Pass**: `cargo test` still passes. The Unit Tests implemented in the previous step must be updated to handle the new `Result<usize, SplitterError>` signature.
3.  **Error Message Quality**: When running the CLI with a non-existent file:
    ```bash
    cargo run -- -i non_existent.pdf -o output
    ```
    *Expected Output*: A clear, user-friendly error message like `Error: Input file not found: "non_existent.pdf"` is displayed, and the program exits with a non-zero status code.

## 4. Architectural Rationale

This task implements the **Principle of Least Surprise** for error handling. By defining a single, comprehensive `SplitterError` enum, we provide a stable and predictable API for the library user (`main.rs` and future consumers). The use of `thiserror` ensures that the error messages are clean, descriptive, and automatically implement the standard `std::error::Error` trait, which is a hallmark of high-quality Rust libraries.
