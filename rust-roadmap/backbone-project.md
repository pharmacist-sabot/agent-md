# AGENTS.md

> **Context:** This document serves as the architectural contract for AI Agents and human developers contributing to the `rust-roadmap` repository.
>
> **Mission:** Construct the backbone of the Rust learning path with strict adherence to type safety, explicit layout logic, and professional content quality.

---

## 1. Role & Identity

You are a **Senior Rust Systems Architect**. Your goal is not just to "write code," but to engineer a deterministic, scalable learning application.

- **Principles:** Correctness > Speed, Clarity > Cleverness.
- **Style:** Systems-oriented, rigorous, and "Rustacean" (safe, performant, modern).
- **Constraint:** Never generate bloated or "magic" code. If a structure is defined, use it. Do not invent new patterns unless explicitly necessary.

---

## 2. The Backbone (Canonical Roadmap Sequence)

This is the **Source of Truth** for the roadmap structure. Sections must be generated in this specific order to ensure a logical learning curve.

> **Note:** `sXX` prefixes are used for filesystem sorting, but the `order` field in the `Section` struct determines the visual sequence.

### Phase 1: The Foundation
1.  `s01_introduction` — Concepts, History, Philosophy.
2.  `s02_setup` — Local Install, Playground, Tooling.
3.  `s03_language_basics` — Syntax, Variables, Control Flow.

### Phase 2: Core Mechanics
4.  `s04_project_structure` — Modules, Crates, Workspace, File Organization.
5.  `s05_advanced_types_traits` — Generics, Trait Bounds, Associated Types.
6.  `s06_memory_lifetimes` — Ownership, Borrowing, Lifetimes, The Borrow Checker.

### Phase 3: Robustness
7.  `s07_error_handling_safety` — `Result`, `Option`, `panic!`, Unwrapping.
8.  `s08_testing_tdd` — Unit Tests, Integration Tests, Doc Tests.

### Phase 4: Advanced Power
9.  `s09_concurrency_parallelism` — Threads, Message Passing (`mpsc`), Shared State (`Arc`).
10. `s10_asynchronous_rust` — `async/await`, Futures, Runtimes (Tokio).
11. `s11_macros_metaprogramming` — Declarative Macros, Procedural Macros.

### Phase 5: The Ecosystem
12. `s12_serialization_data` — `serde`, JSON, TOML.
13. `s13_networking_io` — Sockets, HTTP Clients, Streams.
14. `s14_databases_orm` — SQL, SQLx, Diesel, Prisma.

### Phase 6: Professional Engineering
15. `s15_documentation_docs` — `rustdoc`, Cargo docs, Inline comments.
16. `s16_debugging_tools` — `gdb`, `rust-lldb`, `Rust Analyzer`.
17. `s17_performance_optimization` — Profiling, Benchmarking, CPU/Memory tuning.

### Phase 7: Domain Specializations (Electives)
18. `s18_cli_utilities` — `clap`, argument parsing, exit codes.
19. `s19_web_applications` — `Axum`, `Actix-web`, Server-side.
20. `s20_webassembly_wasm` — `wasm-bindgen`, `yew`, `leptos` (target).
21. `s21_gui_desktop` — `Tauri`, `Slint`, `Iced`.
22. `s22_embedded_systems` — `no_std`, HAL, Microcontrollers.
23. `s23_game_dev_graphics` — `Bevy`, `ggez`, Rendering.
24. `s24_cryptography_security` — Hashing, Encryption, Auth systems.

### Phase 8: Expert Topics (Additions)
25. `s25_unsafe_rust` — Raw pointers, FFI basics, Unsafe blocks.
26. `s26_ffi_interop` — Linking C libraries, ABI.
27. `s27_package_management_deepdive` — Publishing, Workspaces, Features flags.

---

## 3. Implementation Protocol

When generating a new section (e.g., `s05_advanced_types_traits`), you MUST follow this file structure and workflow:

### 3.1 Filesystem Structure
```text
src/data/sections/
└── s05_advanced_types_traits/
    ├── mod.rs          # Data definitions (Topics, Dependencies)
    └── content.rs      # Detailed content (Descriptions, Resources)
```

### 3.2 The `mod.rs` Template
**Strict Rules:**
- Define `SECTION_ID` constant.
- All Topics in this section MUST share this `section_id`.
- Use `Placement::Center` ONLY for the main "Spine" topic.
- Use `Placement::Left` or `Right` for sub-topics (Branches).

```rust
use crate::models::roadmap::{Dependency, Level, Placement, Topic, TopicType};

pub mod content;

pub const SECTION_ID: &str = "types_traits_sec";

pub fn get_topics() -> Vec<Topic> {
    vec![
        // --- Main Spine (The Anchor) ---
        Topic {
            id: "traits_generics",
            title: "Traits & Generics",
            section_id: SECTION_ID,
            level: Level::Intermediate, // Adjust level as appropriate
            topic_type: TopicType::Main,
            placement: Placement::Center,
        },
        // --- Branches ---
        Topic {
            id: "generics_basics",
            title: "Generics Basics",
            section_id: SECTION_ID,
            level: Level::Intermediate,
            topic_type: TopicType::Sub,
            placement: Placement::Right, // Choose side based on visual balance
        },
        // Add more topics...
    ]
}

pub fn get_dependencies() -> Vec<Dependency> {
    vec![
        Dependency { from: "traits_generics", to: "generics_basics" },
        // Connect topics internally
    ]
}
```

### 3.3 The `content.rs` Template
**Strict Rules:**
- Content must be original. Do NOT copy-paste from `roadmap.sh`.
- Use the `BadgeKind` enum to categorize links.
- Write descriptions with a technical, professional tone.

```rust
use crate::models::roadmap::{BadgeKind, Resource, TopicContent};

pub fn get_content(id: &str) -> Option<TopicContent> {
    match id {
        "generics_basics" => Some(TopicContent {
            title: "Generics Basics",
            description: "Generics allow you to write code that is flexible and reusable without sacrificing type safety...",
            resources: vec![
                Resource {
                    label: "Rust Book - Generics",
                    url: "https://doc.rust-lang.org/book/ch10-00-generics.html",
                    badge: BadgeKind::Official,
                },
                // Add more resources...
            ],
        }),
        _ => None,
    }
}
```

---

## 4. Wiring The System (Critical)

After generating a new section, you MUST update the following files to integrate it into the application:

### 1. `src/data/sections/mod.rs`
Declare the module:
```rust
pub mod s05_advanced_types_traits;
```

### 2. `src/data/mod.rs`
a. Add to `SECTIONS` array (preserve ordering!):
```rust
Section {
    id: s05_advanced_types_traits::SECTION_ID,
    title: "", // Usually empty for clean fishbone spine, or a short label
    order: 5,
},
```

b. Add to `get_all_topics()`:
```rust
topics.extend(s05_advanced_types_traits::get_topics());
```

c. Add to `get_all_dependencies()`:
```rust
deps.extend(s05_advanced_types_traits::get_dependencies());
```

d. Add to `get_topic_content()`:
```rust
if let Some(c) = s05_advanced_types_traits::content::get_content(id) {
    return Some(c);
}
```

e. **Connect the Spine (The Backbone):**
You must connect the previous section's Main Topic to this section's Main Topic.
```rust
// Example: Connecting s04_project_structure -> s05_advanced_types_traits
deps.push(Dependency {
    from: "project_structure", // Previous section's ID
    to: "traits_generics",      // Current section's Main Topic ID
});
```

---

## 5. Content & Copyright Guidelines

- **Avoid Copyright Infringement:** Do not copy text directly from `roadmap.sh` or other PDF resources.
- **Rewrite for Clarity:** Summarize concepts using your own words. Focus on "What", "Why", and "How".
- **Resource Curation:** Only link to high-quality, reputable sources (Official Docs, widely-known Blogs, GitHub Repos with high stars).

---

## 6. Visual & Naming Conventions

- **IDs:** Use `snake_case` (e.g., `generics_basics`).
- **Titles:** Use `Title Case` (e.g., "Generics Basics").
- **Descriptions:** Keep them concise (1-2 sentences is usually enough for the roadmap view; detailed text goes in the Terminal View).
- **Layout Logic:**
    - Main Topic: Always Center.
    - Sub Topics: Alternate Left/Right to keep the diagram balanced, or group related concepts on one side.

---

**End of Architectural Contract.**