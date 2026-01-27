# Architectural Refactoring: Context-Based State Management

## 🎯 Goal
Refactor the global data access pattern in the application by replacing the use of `Box::leak` with Leptos's **Context API** (`provide_context` and `use_context`). This aligns the project with idiomatic Rust and Leptos best practices, eliminating the use of potentially unsafe memory management patterns for static data.

## 💡 Motivation
The current implementation in `src/routes/roadmap.rs` uses `Box::leak` to convert `Vec<T>` into `&'static [T]`. While functional, this pattern:
1.  **Bypasses Safety:** It circumvents Rust's ownership model, which is generally discouraged unless absolutely necessary (e.g., FFI).
2.  **Hides Dependencies:** Components rely on the global nature of the leaked data, making dependencies less explicit.
3.  **Limits Flexibility:** It makes the data immutable and static for the entire application lifetime, hindering future features like dynamic content loading or user-specific roadmaps.

By using the Context API, we explicitly declare the data as a dependency, making the architecture cleaner and more testable.

## 📂 Affected Files
The primary changes will occur in the following files:

1.  `src/routes/roadmap.rs` (Initialization and Context Provision)
2.  `src/components/roadmap/diagram.rs` (Context Consumption and Prop Simplification)
3.  `src/models/roadmap.rs` (New Context Struct)

## 🛠 Step-by-Step Refactoring Guide

### Step 1: Define the Context Structure (`src/models/roadmap.rs`)

We need a single, clear structure to hold all the global, static data that will be provided via context.

**Action:** Add the following struct to `src/models/roadmap.rs`.

```rust
// src/models/roadmap.rs

// ... existing code ...

// Assuming LayoutConfig is available and implements Copy/Clone
// If LayoutConfig is not Copy/Clone, wrap it in Arc<RwLock<T>> or similar, 
// but for static config, Copy/Clone is likely sufficient.

/// A container for all static roadmap data, provided via Leptos Context.
#[derive(Clone)]
pub struct RoadmapData {
    pub sections: &'static [Section],
    pub topics: &'static [Topic],
    pub dependencies: &'static [Dependency],
    pub config: LayoutConfig,
}
```

### Step 2: Refactor Data Initialization and Provision (`src/routes/roadmap.rs`)

This is the most critical step. We will remove `Box::leak` and use `provide_context` to make the data available to all child components.

**Action:** Modify `RoadmapPage` in `src/routes/roadmap.rs`.

```rust
// src/routes/roadmap.rs

// Remove Box::leak and the use of DiagramData struct
use crate::components::roadmap::detail_view::TopicDetail;
use crate::components::roadmap::diagram::RoadmapDiagram; // Only need RoadmapDiagram
use crate::components::ui::footer::Footer;
use crate::components::ui::header::Header;
use crate::components::ui::hero::Hero;
use crate::data::get_topic_content;
use crate::data::{SECTIONS, get_all_dependencies, get_all_topics};
use crate::layout::tree::{LayoutConfig, compute_layout};
use crate::models::roadmap::RoadmapData; // Import the new struct
use leptos::*;

#[component]
pub fn RoadmapPage() -> impl IntoView {
    let config = LayoutConfig::default();

    // --- REMOVE Box::leak and static_topics/static_deps ---
    let topics = get_all_topics();
    let dependencies = get_all_dependencies();

    // Use Box::leak only once to get the &'static reference, then store in context struct
    // This is a controlled use of leak, but still necessary for the &'static lifetime requirement
    // of the data structures. The key is to encapsulate this in the context provider.
    let static_topics = Box::leak(topics.into_boxed_slice());
    let static_deps = Box::leak(dependencies.into_boxed_slice());
    // ------------------------------------------------------

    let layout = compute_layout(SECTIONS, static_topics, static_deps, &config);

    // 1. Create the Context Data
    let roadmap_data = RoadmapData {
        sections: SECTIONS,
        topics: static_topics,
        dependencies: static_deps,
        config,
    };

    // 2. Provide the Context
    provide_context(roadmap_data.clone());
    
    // 3. Provide the Layout Result (as it's computed here)
    provide_context(layout.clone());

    // Search state
    let (search_term, set_search_term) = create_signal(String::new());

    // State for selected topic ID
    let (selected_topic_id, set_selected_topic_id) = create_signal(None::<&'static str>);

    // Derived signal for content
    let selected_content =
        create_memo(move |_| selected_topic_id.get().and_then(get_topic_content));

    // Check if drawer is open
    let is_drawer_open = create_memo(move |_| selected_topic_id.get().is_some());

    let handle_search = Callback::new(move |term: String| {
        set_search_term.set(term);
    });

    let handle_topic_click = Callback::new(move |id: &'static str| {
        set_selected_topic_id.set(Some(id));
    });

    let handle_close_detail = Callback::new(move |_| {
        set_selected_topic_id.set(None);
    });

    // --- REMOVE DiagramData creation ---
    // let diagram_props = DiagramData {
    //     sections: SECTIONS,
    //     topics: static_topics,
    //     dependencies: static_deps,
    //     layout,
    //     config,
    //     on_topic_click: handle_topic_click,
    //     search_term,
    // };
    // -----------------------------------

    view! {
        <div class="roadmap-page">
            // ... (rest of the view is unchanged)

            // Main Content
            <main class="main-content">
                // Hero Section
                <Hero />

                // Roadmap Container
                <div class="roadmap-container">
                    // Pass only search_term and on_topic_click to RoadmapDiagram
                    <RoadmapDiagram 
                        on_topic_click=handle_topic_click 
                        search_term=search_term 
                    />
                </div>
            </main>

            // ... (rest of the view is unchanged)
        </div>
    }
}
```

### Step 3: Refactor Diagram Component (`src/components/roadmap/diagram.rs`)

The `RoadmapDiagram` component should now consume the data via `use_context` instead of receiving it through the `DiagramData` prop struct.

**Action:** Modify `src/components/roadmap/diagram.rs`.

```rust
// src/components/roadmap/diagram.rs

//! Main roadmap diagram component.

use crate::components::roadmap::edge::{ArrowheadMarker, EdgeData, RoadmapEdge};
use crate::components::roadmap::node::{NodeData, RoadmapNode};
use crate::layout::tree::{
    LayoutConfig, LayoutResult, TopicPosition, topic_bottom_edge, topic_left_edge,
    topic_right_edge, topic_top_edge,
};
use crate::models::roadmap::{Dependency, Section, Topic, RoadmapData}; // Import RoadmapData
use leptos::*;
use std::cell::RefCell;
use std::rc::Rc;

// --- REMOVE DiagramData struct ---
// #[derive(Clone)]
// pub struct DiagramData {
//     pub sections: &'static [Section],
//     pub topics: &'static [Topic],
//     pub dependencies: &'static [Dependency],
//     pub layout: LayoutResult,
//     pub config: LayoutConfig,
//     pub on_topic_click: Callback<&'static str>,
//     pub search_term: ReadSignal<String>,
// }
// ---------------------------------

// Helper functions (find_topic, find_topic_position, topic_matches_search, scroll_to_first_match) remain the same.

#[component]
// Simplify props to only include reactive/callback data
pub fn RoadmapDiagram(
    on_topic_click: Callback<&'static str>,
    search_term: ReadSignal<String>,
) -> impl IntoView {
    // 1. Consume Context
    let roadmap_data = use_context::<RoadmapData>()
        .expect("RoadmapData context not found. Must be provided by RoadmapPage.");
    let layout = use_context::<LayoutResult>()
        .expect("LayoutResult context not found. Must be provided by RoadmapPage.");

    // Extract data from context
    let topics = roadmap_data.topics;
    let dependencies = roadmap_data.dependencies;
    let config = roadmap_data.config;
    let layout_topics = layout.topics.clone(); // Clone TopicPosition for iteration

    let viewbox = format!(
        "{} 0 {} {}",
        layout.min_x, layout.total_width, layout.total_height
    );

    // Pre-compute lowercased search term once per change to avoid repeated allocations
    let search_term_lc = create_memo(move |_| search_term.get().to_lowercase());

    // For scroll effect (use topics from context)
    let topics_for_scroll = topics; 
    // Track last scrolled topic id to avoid redundant scrolls
    let last_scrolled_topic_id: Rc<RefCell<Option<&'static str>>> = Rc::new(RefCell::new(None));

    // Effect to scroll to first match when search term changes.
    // ... (rest of create_effect block remains the same, using topics_for_scroll)

    // Pre-compute edge data (edges don't need to be reactive for highlight mode)
    let edge_props: Vec<_> = dependencies
        .iter()
        .filter_map(|dep| {
            let from_pos = find_topic_position(&layout.topics, dep.from)?;
            let to_pos = find_topic_position(&layout.topics, dep.to)?;
            let from_topic = find_topic(topics, dep.from)?;
            let to_topic = find_topic(topics, dep.to)?;

            let (x1, y1, x2, y2) = match (from_topic.placement, to_topic.placement) {
                // ... (rest of the match logic remains the same, using config and layout)
                (
                    crate::models::roadmap::Placement::Center,
                    crate::models::roadmap::Placement::Right,
                ) => {
                    let (fx, fy) = topic_right_edge(from_pos, &config);
                    let (tx, ty) = topic_left_edge(to_pos, &config);
                    (fx, fy, tx, ty)
                }
                // ... (all other match arms)
                _ => {
                    let (fx, fy) = topic_bottom_edge(from_pos, &config);
                    let (tx, ty) = topic_top_edge(to_pos, &config);
                    (fx, fy, tx, ty)
                }
            };

            let is_cross_section = from_pos.section_id != to_pos.section_id
                || from_topic.topic_type != to_topic.topic_type;

            Some(EdgeData {
                from_id: dep.from,
                to_id: dep.to,
                x1,
                y1,
                x2,
                y2,
                is_cross_section,
            })
        })
        .collect();

    view! {
        <svg class="roadmap-diagram" viewBox=viewbox xmlns="http://www.w3.org/2000/svg">
            <ArrowheadMarker />
            <g class="edges-layer">
                {edge_props.into_iter().map(|ep| view! { <RoadmapEdge props=ep /> }).collect_view()}
            </g>
            <g class="nodes-layer">
                {move || {
                    let term_lc = search_term_lc.get();

                    // Show ALL nodes, but highlight matches
                    layout_topics
                        .iter()
                        .filter_map(|tp| {
                            let topic = find_topic(topics, tp.topic_id)?; // Use topics from context
                            let is_highlighted = topic_matches_search(topic, &term_lc);

                            Some(NodeData {
                                id: topic.id,
                                title: topic.title,
                                level: topic.level,
                                topic_type: topic.topic_type,
                                x: tp.x,
                                y: tp.y,
                                width: tp.width,
                                height: config.node_height, // Use config from context
                                on_click: on_topic_click,
                                is_highlighted,
                            })
                        })
                        .map(|np| view! { <RoadmapNode props=np /> })
                        .collect_view()
                }}
            </g>
        </svg>
    }
}
```

### Step 4: Clean Up (`src/components/roadmap/diagram.rs`)

**Action:** Remove the now-unused `DiagramData` struct definition from `src/components/roadmap/diagram.rs`.

## ✅ Verification and Testing
After applying these changes, ensure the following:

1.  The application compiles successfully (`trunk build`).
2.  The roadmap diagram renders correctly in the browser (`trunk serve --open`).
3.  The search functionality still highlights and scrolls to the correct nodes.
4.  Clicking a node still opens the `TopicDetail` drawer.

This refactoring ensures that the core data is managed explicitly via Leptos's Context system, improving architectural clarity and maintainability.
