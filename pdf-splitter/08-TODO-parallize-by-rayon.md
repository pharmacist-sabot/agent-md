# Parallelize Page Processing with `rayon`

## 1. Overview

This task is a critical performance optimization step. We will introduce the **`rayon`** library to parallelize the core PDF splitting loop within `PdfSplitter::split_pdf`. This will allow the application to utilize multiple CPU cores simultaneously, drastically reducing the processing time for large PDF documents. The refactoring must be done safely, adhering to Rust's ownership and thread-safety guarantees.

## 2. Technical Specifications

### 2.1. Dependency Update

Update `Cargo.toml` to include `rayon` as a dependency.

```toml
[dependencies]
lopdf = "0.39.0"
clap = { version = "4.5", features = ["derive"] }
thiserror = "1.0"
rayon = "1.10" # New dependency
```

### 2.2. Refactor Core Splitting Logic (`src/lib.rs`)

The main change involves replacing the standard sequential iteration (`for page_number in &all_page_numbers`) with a parallel iteration (`all_page_numbers.par_iter().try_for_each(...)`).

**Required Code Structure in `src/lib.rs`:**

1.  **Import `rayon`:** Add the necessary trait import.
    ```rust
    use rayon::prelude::*; // Add this line
    // ... other imports
    ```

2.  **Refactor the Loop in `PdfSplitter::split_pdf`:** The sequential `for` loop must be replaced with `par_iter().try_for_each()`.

    **Goal:** The parallel loop must handle errors correctly. `try_for_each` is used because the inner logic can return a `Result<_, SplitterError>`.

    ```rust
    // ... inside PdfSplitter::split_pdf ...
    
    // ... setup code ...
    
    // Get all page numbers from the document
    let pages = doc.get_pages();
    let total_pages = pages.len();
    let all_page_numbers: Vec<u32> = pages.keys().copied().collect();

    // --- NEW PARALLEL LOGIC ---
    // Use par_iter() to process pages in parallel.
    // try_for_each is used to handle the Result<_, SplitterError> returned by the closure.
    all_page_numbers.par_iter().try_for_each(|&page_number| {
        // NOTE: The inner logic must be thread-safe.
        // 1. doc.clone() is safe because it creates a new Document instance for each thread.
        // 2. new_doc.save() is safe because each thread writes to a unique file path.
        
        // NOTE: Console output (println!) should be removed from the library layer
        // as it can lead to interleaved output in parallel execution.
        // The previous task should have already removed this, but ensure it is gone.
        // If not, remove: println!("Processing page {}/{}...", page_number, total_pages);

        // Create a new document for each page by cloning the original
        let mut new_doc = doc.clone();

        // Collect page numbers to delete (all pages except current)
        let pages_to_delete: Vec<u32> = all_page_numbers
            .iter()
            .filter(|&&p| p != page_number)
            .copied()
            .collect();

        // Remove unwanted pages
        new_doc.delete_pages(&pages_to_delete);

        // Clean up unused objects and renumber for a smaller file size
        new_doc.prune_objects();
        new_doc.renumber_objects();

        // Save the single-page document (using secure path handling from previous task)
        let filename = format!("page_{:03}.pdf", page_number);
        let output_path = output_dir.join(filename);

        // Use the ? operator to propagate SplitterError
        new_doc.save(&output_path).map_err(SplitterError::Pdf)?;
        
        // Return Ok(()) for try_for_each
        Ok(())
    })?; // The final ? propagates any error from the parallel execution

    // ... rest of the function (return Ok(total_pages)) ...
    ```

### 2.3. Clean Up Console Output

**Crucial Step for Parallelization:** Ensure that any `println!` or `eprintln!` calls that were previously inside the loop are removed from `src/lib.rs`. Parallel execution makes the order of these outputs non-deterministic and can clutter the console. The CLI layer (`src/main.rs`) should handle all user-facing progress reporting.

**Action:**
1.  Verify and remove the line `println!("Processing page {}/{}...", page_number, total_pages);` from `PdfSplitter::split_pdf` in `src/lib.rs`.
2.  The CLI layer (`src/main.rs`) should be updated to provide a single, final success message after the `split_pdf` call completes.

## 3. Verification

The implementation is considered complete when the following conditions are met:

1.  **Successful Compilation**: `cargo build` completes without warnings or errors.
2.  **Test Pass**: `cargo test` still passes. The Unit Tests confirm that the parallel logic produces the exact same, correct result as the sequential logic.
3.  **Performance Check (Conceptual)**: When processing a large PDF (e.g., 100+ pages), the execution time should be significantly reduced compared to the sequential version, demonstrating effective parallelization.

## 4. Architectural Rationale

This optimization leverages the **Data Parallelism** model, which is highly effective for tasks where the work for each item (page) is independent. By using `rayon`, we gain:

*   **Automatic Thread Management**: `rayon` handles the thread pool and workload distribution efficiently.
*   **Safety**: `rayon` is designed to work within Rust's safety rules, preventing common concurrency bugs like data races.
*   **Scalability**: The application's performance will automatically scale with the number of available CPU cores, a key feature of production-grade software.