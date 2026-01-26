# Refactor Core Logic into Library Crate (`src/lib.rs`)

## 1. Overview

This task focuses on the critical architectural refactoring of the `pdf-splitter` utility. We will implement the **Separation of Concerns** principle by moving all PDF processing logic from the CLI entry point (`src/main.rs`) into a dedicated **Library Crate** (`src/lib.rs`). This establishes a clear **Service Layer** pattern, making the core functionality reusable, testable, and independent of the I/O and CLI implementation.

## 2. Technical Specifications

### 2.1. Create Library Crate (`src/lib.rs`)

Create a new file `src/lib.rs` and define the main service struct, `PdfSplitter`.

**Required Code Structure in `src/lib.rs`:**

```rust
use lopdf::Document;
use std::path::{Path, PathBuf};
use std::fs;

// --- 1. Define the Core Service Struct ---
/// A service responsible for loading, splitting, and saving PDF documents.
pub struct PdfSplitter;

impl PdfSplitter {
    /// Initializes the PdfSplitter service.
    pub fn new() -> Self {
        PdfSplitter {}
    }

    // --- 2. Define the Core Splitting Method ---
    /// Loads a PDF from the input path and splits it into individual pages,
    /// saving them to the specified output directory.
    ///
    /// This method encapsulates the entire business logic of the application.
    pub fn split_pdf(
        &self,
        input_path: &Path,
        output_dir: &Path,
    ) -> lopdf::Result<usize> {
        // 1. Input Validation (Moved from main.rs)
        if !input_path.exists() {
            // NOTE: In the next phase (Error Handling), this will be replaced by a custom error.
            // For now, we use a simple error return or panic to maintain the current error flow.
            // For this refactoring step, we will assume the caller (main.rs) handles the initial check
            // or we return a lopdf::Error if the load fails.
            // For robustness, we will keep the load logic here and let lopdf handle the error.
        }

        // Create the output directory (Moved from main.rs)
        fs::create_dir_all(output_dir)?;

        // Load the source PDF document (Moved from main.rs)
        let doc = Document::load(input_path)?;

        // Get all page numbers from the document (Moved from main.rs)
        let pages = doc.get_pages();
        let total_pages = pages.len();
        let all_page_numbers: Vec<u32> = pages.keys().copied().collect();

        // Loop and split logic (Moved from main.rs)
        for page_number in &all_page_numbers {
            // NOTE: Console output (println!) should ideally be removed from the core library
            // and handled by the CLI layer (main.rs). For this refactoring step, we will
            // temporarily keep the println! calls here, but mark them for removal in the
            // next phase (CLI UX Refinement).
            println!("Processing page {}/{}...", page_number, total_pages);

            let mut new_doc = doc.clone();

            let pages_to_delete: Vec<u32> = all_page_numbers
                .iter()
                .filter(|&&p| p != *page_number)
                .copied()
                .collect();

            new_doc.delete_pages(&pages_to_delete);
            new_doc.prune_objects();
            new_doc.renumber_objects();

            // Save the single-page document (Moved from main.rs)
            let output_filename = format!(
                "{}/page_{:03}.pdf",
                output_dir.to_str().unwrap(),
                page_number
            );
            new_doc.save(&output_filename)?;
        }

        // Return the number of pages split
        Ok(total_pages)
    }
}
```

### 2.2. Update CLI Entry Point (`src/main.rs`)

The `src/main.rs` file must be reduced to its role as a simple **CLI Frontend**. It should handle argument parsing (from the previous task) and then call the `PdfSplitter` service.

**Required Code Structure in `src/main.rs`:**

```rust
use clap::Parser;
use std::path::PathBuf;
use pdf_splitter::PdfSplitter; // Import the new service

// Re-define or ensure the Args struct from the previous task is present
#[derive(Parser, Debug)]
#[command(author, version, about, long_about = None)]
struct Args {
    #[arg(short, long, value_name = "FILE")]
    input: PathBuf,

    #[arg(short, long, value_name = "DIR", default_value = "output_pages")]
    output: PathBuf,
}

fn main() -> lopdf::Result<()> {
    let args = Args::parse();
    let input_path = &args.input;
    let output_dir = &args.output;

    // --- NEW LOGIC: CLI Frontend ---

    // 1. Input Validation (Moved from lib.rs to the CLI layer)
    if !input_path.exists() {
        eprintln!("Error: PDF file not found at '{}'", input_path.display());
        eprintln!("Please ensure the input file exists.");
        std::process::exit(1);
    }

    println!("Starting to split: {}", input_path.display());

    // 2. Initialize and Call the Core Service
    let splitter = PdfSplitter::new();
    let total_pages = splitter.split_pdf(input_path, output_dir)?;

    // 3. Output/Success Message (Moved from lib.rs to the CLI layer)
    println!(
        "\n✅ Successfully split {} pages into '{}'",
        total_pages,
        output_dir.display()
    );

    Ok(())
}
```

## 3. Verification

The implementation is considered complete when the following conditions are met:

1.  **Successful Compilation**: `cargo build` completes without warnings or errors.
2.  **Functional Equivalence**: The program runs exactly as it did before the refactoring, but now using the `PdfSplitter` service.
    ```bash
    # Assuming a test.pdf exists
    cargo run -- -i /path/to/test.pdf -o /tmp/split_output
    ```
    *Expected Output*: The PDF is split, and the success message is displayed.
3.  **Code Structure**: The file `src/main.rs` contains only the `Args` struct, the `main` function, and minimal I/O/CLI logic. All PDF manipulation logic resides in `src/lib.rs`.

## 4. Architectural Rationale

This refactoring establishes a **Clean Architecture** boundary:

| Component | Role | Responsibility |
| :--- | :--- | :--- |
| `src/lib.rs` (`PdfSplitter`) | **Core Service Layer** | Business Logic: Loading PDF, iterating pages, cloning, pruning, and saving individual pages. **Must be independent of I/O.** |
| `src/main.rs` | **CLI Frontend/Adapter** | Infrastructure Logic: Parsing CLI arguments, handling initial file existence check, displaying user messages (`println!`, `eprintln!`), and calling the core service. |

This separation is crucial for the next step: **Unit Testing** the `PdfSplitter` without needing to mock file system operations or CLI interactions.
