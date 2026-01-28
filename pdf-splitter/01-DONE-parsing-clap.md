# Implement Robust CLI Argument Parsing with `clap`

## 1. Overview

This task is the first critical step in transforming the `pdf-splitter` utility from a hardcoded prototype into a production-grade Command-Line Interface (CLI) tool. We will replace the current hardcoded input and output paths in `src/main.rs` with dynamic, user-provided arguments using the industry-standard Rust library, **`clap`** (Command Line Argument Parser).

The implementation must adhere to world-class standards, focusing on **Type Safety**, **Robust Error Handling**, and **Clean Code Architecture**.

## 2. Technical Specifications

### 2.1. Dependency Update

Update `Cargo.toml` to include the `clap` library with the `derive` feature for a declarative and clean argument definition.

```toml
[dependencies]
lopdf = "0.39.0"
clap = { version = "4.5", features = ["derive"] }
```

### 2.2. CLI Structure Definition (`src/main.rs`)

Define the CLI structure using the `clap::Parser` derive macro. The application should accept two primary arguments:

1.  **Input File Path**: The path to the PDF file to be split.
2.  **Output Directory Path**: The directory where the split pages will be saved.

**Required Code Structure in `src/main.rs`:**

```rust
use clap::Parser;
use std::path::PathBuf;

/// A simple, fast command-line tool to split PDF documents into individual pages.
///
/// This utility takes an input PDF file and splits it into multiple single-page
/// PDF files, saving them to a specified output directory.
#[derive(Parser, Debug)]
#[command(author, version, about, long_about = None)]
struct Args {
    /// Path to the input PDF file to be split.
    /// Example: /path/to/document.pdf
    #[arg(short, long, value_name = "FILE")]
    input: PathBuf,

    /// Path to the output directory where split pages will be saved.
    /// Defaults to "output_pages" in the current working directory.
    /// Example: /path/to/output/
    #[arg(short, long, value_name = "DIR", default_value = "output_pages")]
    output: PathBuf,
}

fn main() -> lopdf::Result<()> {
    let args = Args::parse();
    // ... rest of the main logic will use args.input and args.output
    // ... (The existing logic should be adapted to use these variables)
    
    // Example of usage:
    // let input_path = &args.input;
    // let output_dir = &args.output;
    
    // ...
    
    Ok(())
}
```

### 2.3. Refactoring the `main` Function

The existing hardcoded paths must be replaced with the values from the parsed `Args` struct.

**Key Changes in `src/main.rs`:**

1.  **Parsing**: Call `Args::parse()` at the beginning of `main`.
2.  **Input Path**: Replace `let input_path = Path::new("input/sample.pdf");` with `let input_path = &args.input;`.
3.  **Output Path**: Replace `let output_dir = Path::new("output_pages");` with `let output_dir = &args.output;`.
4.  **Path Safety**: The use of `std::path::PathBuf` for argument types ensures that the paths are handled safely and correctly by `clap`, aligning with Rust's best practices for I/O.

### 2.4. Architectural Pre-Requisite (Zero-Downtime Refactoring)

To maintain system stability and prepare for the next phase (Modularity), the developer **MUST NOT** introduce any new core logic. This task is strictly limited to:

1.  Updating `Cargo.toml`.
2.  Defining the `Args` struct with `clap::Parser`.
3.  Replacing the hardcoded paths in the existing `main` function with the parsed values from `args.input` and `args.output`.

This approach ensures that the system's core functionality remains unchanged while the interface is modernized, minimizing the risk of introducing regressions.

## 3. Verification

The implementation is considered complete when the following command-line executions are successful:

1.  **Help Command**:
    ```bash
    cargo run -- --help
    ```
    *Expected Output*: Displays the help message, version, and descriptions for `-i`/`--input` and `-o`/`--output`.

2.  **Missing Input (Error Handling)**:
    ```bash
    cargo run
    ```
    *Expected Output*: `clap` should automatically display an error message indicating that the required `--input` argument is missing, and exit gracefully.

3.  **Successful Execution**:
    ```bash
    # Assuming a test.pdf exists
    cargo run -- -i /path/to/test.pdf -o /tmp/split_output
    ```
    *Expected Output*: The program runs, splits the PDF, and saves the pages to `/tmp/split_output/`.
