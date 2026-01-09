# AGENTS.md

**Role**: Senior Rust Architect & Content Curator
**Objective**: Enforce strict type safety and architectural consistency when adding content to the Rust Roadmap.
**Language**: Rust (Strict), Markdown

---

## 1. Core Principles

1.  **Type Safety First**: All content must be defined using the provided `Topic`, `Dependency`, and `TopicContent` structs. No magic strings in the rendering logic.
2.  **Explicit Layout**: The roadmap uses a "Fishbone" layout. Every topic must have an explicit `Placement` (`Center`, `Left`, or `Right`). Do not rely on automatic graph algorithms.
3.  **Atomic Modularity**: Each section (e.g., `s01_introduction`) is self-contained. Code for a specific section lives strictly in `src/data/sections/sXX_SECTION_NAME/`.
4.  **Content Hierarchy**:
    *   **Official Docs**: Highest priority.
    *   **RFCs / Specs**: For language design features.
    *   **Blog Posts / Articles**: Only from high-quality sources (e.g., Without Boats, Rust Blog).
    *   **Videos**: Only from official Rust channels or verified experts.

---

## 2. Data Model Definitions

Before adding content, ensure the following types are used correctly.

### The `Topic` Entity
Located in `src/data/sections/sXX_.../mod.rs`.

```rust
pub struct Topic {
    /// Unique identifier (snake_case)
    pub id: &'static str,
    /// Display title in the node
    pub title: &'static str,
    /// The section this topic belongs to (e.g., "s03_language_basics")
    pub section_id: &'static str,
    /// Difficulty level
    pub level: Level,
    /// Main node vs Sub node
    pub topic_type: TopicType,
    /// Visual position on the fishbone spine
    pub placement: Placement,
}
```

**Enums:**
*   **`Placement`**: `Center` (Spine), `Left` (Branch), `Right` (Branch).
*   **`Level`**: `Beginner`, `Intermediate`, `Advanced`.
*   **`TopicType`**: `Main`, `Sub`.

### The `Dependency` Entity
Located in `src/data/sections/sXX_.../mod.rs`.

```rust
pub struct Dependency {
    pub from: &'static str, // ID of the parent topic
    pub to: &'static str,   // ID of the child topic
}
```

### The `TopicContent` Entity
Located in `src/data/sections/sXX_.../content.rs`.

```rust
pub struct TopicContent {
    pub title: &'static str,
    pub description: &'static str,
    pub resources: Vec<Resource>,
}

pub struct Resource {
    pub label: &'static str,
    pub url: &'static str,
    pub badge: BadgeKind,
}
```

**Enums:**
*   **`BadgeKind`**: `Official`, `OpenSource`, `Article`, `Video`, `Feed`, `Other(String)`.

---

## 3. Workflow: Adding New Content

When instructed to add content for a specific topic (e.g., "Smart Pointers"), follow these steps:

### Step 1: Update `mod.rs` (Topological Structure)
1.  Navigate to `src/data/sections/sXX_SECTION_NAME/mod.rs`.
2.  Define the `Topic` struct.
    *   **Constraint**: If it is a major milestone on the main path, `placement` MUST be `Center`.
    *   **Constraint**: If it is a concept related to a main topic, `placement` MUST be `Left` or `Right` based on available space.
3.  Add the topic to the `get_topics()` vector.
4.  Define `Dependency` structs to link it to parent(s).
5.  Add dependencies to `get_dependencies()` vector.

### Step 2: Update `content.rs` (Rich Data)
1.  Navigate to `src/data/sections/sXX_SECTION_NAME/content.rs`.
2.  Add a match arm in `get_content(id: &str)` for the new topic ID.
3.  Construct `TopicContent`:
    *   Write a concise description (1-2 sentences).
    *   Curate a list of `Resource`s.
    *   **Strict Rule**: Apply the correct `BadgeKind`. Official documentation MUST use `BadgeKind::Official`.

---

## 4. Layout Logic (Fishbone Constraints)

The layout engine (`src/layout/tree.rs`) expects specific patterns. **Do not deviate.**

*   **Spine Flow**: Main topics generally flow vertically (`Center`).
*   **Branching**:
    *   Sub-topics extending from a `Center` topic can be `Left` or `Right`.
    *   Avoid crossing edges if possible.
*   **Node Dimensions**: The layout config (in `src/components/roadmap/diagram.rs`) calculates coordinates based on fixed widths/heights defined in `LayoutConfig`. Do not manually set X/Y coordinates in the `Topic` struct; the layout engine handles this based on `Placement`.

---

## 5. Code Template

Use this template when generating code for a new topic:

**In `mod.rs`:**
```rust
// 1. Define the Topic
pub const my_new_topic: Topic = Topic {
    id: "smart_pointers",
    title: "Smart Pointers",
    section_id: "s05_advanced_types",
    level: Level::Intermediate,
    topic_type: TopicType::Main,
    placement: Placement::Center, // Explicit placement
};

// 2. Define Dependencies
pub const deps_smart_pointers: [Dependency; 1] = [
    Dependency {
        from: "ownership_basics",
        to: "smart_pointers",
    },
];

// 3. Export
pub fn get_topics() -> Vec<Topic> {
    vec![
        // ... existing topics
        my_new_topic,
    ]
}

pub fn get_dependencies() -> Vec<Dependency> {
    let mut deps = vec![/* ... existing deps */];
    deps.extend(deps_smart_pointers);
    deps
}
```

**In `content.rs`:**
```rust
pub fn get_content(id: &str) -> Option<TopicContent> {
    match id {
        "smart_pointers" => Some(TopicContent {
            title: "Smart Pointers",
            description: "Data structures that act like pointers but have additional metadata and capabilities.",
            resources: vec![
                Resource {
                    label: "The Rust Book - Smart Pointers",
                    url: "https://doc.rust-lang.org/book/ch15-00-smart-pointers.html",
                    badge: BadgeKind::Official,
                },
                Resource {
                    label: "Rust By Example - Box",
                    url: "https://doc.rust-lang.org/rust-by-example/std/box.html",
                    badge: BadgeKind::Official,
                }
            ],
        }),
        _ => None,
    }
}
```

---

## 6. Validation Checklist (Pre-Commit)

Before finalizing the generated code, verify:
- [ ] All IDs are unique and snake_case.
- [ ] `placement` is explicitly set (not inferred).
- [ ] `BadgeKind` is accurate (Official docs vs. Blog).
- [ ] Dependencies form a DAG (no cycles).
- [ ] Section ID matches the file directory name (e.g., `s05_...`).
- [ ] Syntax is valid Rust (Edition 2024).