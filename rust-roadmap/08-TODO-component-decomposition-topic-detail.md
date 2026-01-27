# Component Decomposition: Enhancing Maintainability of TopicDetail

## 🎯 Goal
Decompose the monolithic `TopicDetail` component (`src/components/roadmap/detail_view.rs`) into smaller, reusable, and more maintainable sub-components. This adheres to the principle of **Single Responsibility Principle (SRP)**, making the UI easier to test, understand, and evolve.

## 💡 Motivation
The current `TopicDetail` component handles:
1.  Drawer state and animation logic.
2.  Header/Title rendering.
3.  Description rendering.
4.  Learning Status card rendering and logic.
5.  Resources list rendering.
6.  Code Snippet rendering.

By separating the content sections into dedicated components, we isolate concerns. For instance, the `ResourceList` component will only worry about rendering resources, making it a pure function of its input props.

## 📂 Affected Files
1.  `src/components/roadmap/detail_view.rs` (The main component to be refactored)
2.  `src/components/roadmap/mod.rs` (To export new components)
3.  New files for sub-components (e.g., `src/components/roadmap/topic_resources.rs`, `src/components/roadmap/topic_status.rs`, etc.)

## 🛠 Step-by-Step Implementation Guide

### Step 1: Create New Sub-Component Files

Create the following new files in the `src/components/roadmap/` directory:

- `src/components/roadmap/topic_resources.rs`
- `src/components/roadmap/topic_status.rs`
- `src/components/roadmap/topic_code_preview.rs`

### Step 2: Implement `TopicResources` Component

This component will handle the rendering of the resource list, including the logic for mapping `BadgeKind` to CSS classes.

**Action:** Implement `TopicResources` in `src/components/roadmap/topic_resources.rs`.

```rust
// src/components/roadmap/topic_resources.rs

use leptos::*;
use crate::models::roadmap::{Resource, BadgeKind};

#[component]
pub fn TopicResources(resources: Vec<Resource>) -> impl IntoView {
    view! {
        <div class="drawer__section">
            <h3 class="drawer__section-title">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <polyline points="16 18 22 12 16 6"></polyline>
                    <polyline points="8 6 2 12 8 18"></polyline>
                </svg>
                "Resources"
            </h3>
            <div class="drawer__resources">
                {if resources.is_empty() {
                    view! {
                        <div class="drawer__empty">
                            "No resources listed yet."
                        </div>
                    }.into_view()
                } else {
                    resources.into_iter().map(|res| {
                        let badge_class = match res.badge {
                            BadgeKind::Official => "drawer__resource-badge badge--official",
                            BadgeKind::OpenSource => "drawer__resource-badge badge--opensource",
                            BadgeKind::Crate => "drawer__resource-badge badge--crate",
                            BadgeKind::Article => "drawer__resource-badge badge--article",
                            BadgeKind::Book => "drawer__resource-badge badge--book",
                            // Add other BadgeKind matches here
                            _ => "drawer__resource-badge badge--other",
                        };
                        view! {
                            <a href=res.url target="_blank" rel="noopener noreferrer" class="drawer__resource-item">
                                <span class="drawer__resource-label">{res.label}</span>
                                <span class=badge_class>{format!("{:?}", res.badge)}</span>
                            </a>
                        }
                    }).collect_view()
                }}
            </div>
        </div>
    }
}
```

### Step 3: Implement `TopicStatusCard` Component

This component will handle the "Learning Status" section, including the progress bar and the "Mark as Done" button.

**Action:** Implement `TopicStatusCard` in `src/components/roadmap/topic_status.rs`.

```rust
// src/components/roadmap/topic_status.rs

use leptos::*;

#[component]
pub fn TopicStatusCard() -> impl IntoView {
    // NOTE: In a future refactor, this component should accept a Signal for the status
    // and a Callback for the button click. For now, we keep the static view.
    view! {
        <div class="drawer__section">
            <div class="drawer__status-card">
                <div class="drawer__status-header">
                    <span class="drawer__status-label">"Learning Status"</span>
                    <span class="drawer__status-percent">"0% Complete"</span>
                </div>
                <div class="drawer__progress-bar">
                    <div class="drawer__progress-fill" style="width: 0%"></div>
                </div>
                <button class="drawer__status-button">
                    <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                        <path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"></path>
                        <polyline points="22 4 12 14.01 9 11.01"></polyline>
                    </svg>
                    "Mark as Done"
                </button>
            </div>
        </div>
    }
}
```

### Step 4: Implement `TopicCodePreview` Component

This component will handle the static code snippet display.

**Action:** Implement `TopicCodePreview` in `src/components/roadmap/topic_code_preview.rs`.

```rust
// src/components/roadmap/topic_code_preview.rs

use leptos::*;

#[component]
pub fn TopicCodePreview(title: &'static str) -> impl IntoView {
    view! {
        <div class="drawer__section">
            <h3 class="drawer__section-title">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <polyline points="16 18 22 12 16 6"></polyline>
                    <polyline points="8 6 2 12 8 18"></polyline>
                </svg>
                "Syntax Preview"
            </h3>
            <div class="drawer__code-preview">
                <div class="drawer__code-line">
                    <span class="drawer__code-keyword">"fn"</span>
                    " "
                    <span class="drawer__code-function">"main"</span>
                    "() {"
                </div>
                <div class="drawer__code-line" style="padding-left: 1rem;">
                    <span class="drawer__code-comment">"// This is a standard Rust entry point"</span>
                </div>
                <div class="drawer__code-line" style="padding-left: 1rem;">
                    <span class="drawer__code-macro">"println!"</span>
                    "("
                    <span class="drawer__code-string">{format!("\"Hello, {title}!\"", title=title)}</span>
                    ");"
                </div>
                <div class="drawer__code-line">
                    "}"
                </div>
            </div>
        </div>
    }
}
```

### Step 5: Update `detail_view.rs` (The Parent Component)

The original `TopicDetail` component will now act as a container, importing and composing the new sub-components.

**Action:** Modify `src/components/roadmap/detail_view.rs`.

```rust
// src/components/roadmap/detail_view.rs

use crate::models::roadmap::{BadgeKind, TopicContent};
use leptos::*;

// Import the new sub-components
use super::topic_resources::TopicResources;
use super::topic_status::TopicStatusCard;
use super::topic_code_preview::TopicCodePreview;

#[component]
pub fn TopicDetail(content: TopicContent, on_close: Callback<()>, is_open: bool) -> impl IntoView {
    let drawer_class = if is_open {
        "drawer drawer--open"
    } else {
        "drawer"
    };

    // Get section ID by parsing the content title (simple approach)
    let section_label = "Introduction"; // This could be passed as a prop for accuracy

    view! {
        <div
            class=drawer_class
            role="dialog"
            aria-modal="true"
            aria-labelledby="drawer-title"
        >
            // Drawer Header (Remains here as it manages the drawer's core functionality)
            <div class="drawer__header">
                <div class="drawer__header-content">
                    <div class="drawer__section-label">
                        {section_label}
                    </div>
                    <h1 class="drawer__title" id="drawer-title">
                        {content.title}
                    </h1>
                </div>
                <button
                    class="drawer__close"
                    on:click=move |_| on_close.call(())
                    aria-label="Close drawer"
                >
                    // SVG for close button remains here
                    <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                        <line x1="18" y1="6" x2="6" y2="18"></line>
                        <line x1="6" y1="6" x2="18" y2="18"></line>
                    </svg>
                </button>
            </div>

            // Drawer Body - Now composed of sub-components
            <div class="drawer__body">
                // Description Section (Remains simple enough to keep here)
                <div class="drawer__section">
                    <p class="drawer__description">
                        {content.description}
                    </p>
                </div>

                // Learning Status Section -> TopicStatusCard
                <TopicStatusCard />

                // Resources Section -> TopicResources
                <TopicResources resources=content.resources.clone() />

                // Code Snippet Section -> TopicCodePreview
                <TopicCodePreview title=content.title />
            </div>
        </div>
    }
}
```

### Step 6: Update `mod.rs`

**Action:** Export the new modules in `src/components/roadmap/mod.rs`.

```rust
// src/components/roadmap/mod.rs

pub mod detail_view;
pub mod diagram;
pub mod edge;
pub mod node;
// New exports
pub mod topic_resources;
pub mod topic_status;
pub mod topic_code_preview;
```

## ✅ Verification and Testing
1.  The application compiles successfully (`trunk build`).
2.  The `TopicDetail` drawer opens and displays all content (Header, Description, Status Card, Resources, Code Preview) exactly as before.
3.  The console should not show any errors related to missing components or props.
4.  The new components (`TopicResources`, `TopicStatusCard`, `TopicCodePreview`) are now isolated and ready for dedicated unit testing.
