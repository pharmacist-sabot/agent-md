# Robust Error Handling: Replacing Panics with Graceful Fallbacks

## 🎯 Goal
Refactor code sections that interact with the Browser's Web APIs (via `web-sys`) to replace panic-inducing methods like `unwrap()` and `unwrap_or()` with explicit `Result` and `Option` handling. This ensures the application remains stable and provides meaningful logging when browser operations fail.

## 💡 Motivation
In a production environment, reliance on `unwrap()` for browser operations is a critical vulnerability. Browser APIs can fail for various reasons (e.g., security restrictions, missing elements, unsupported features). A single `unwrap()` failure leads to a panic, crashing the entire WebAssembly application.

By implementing robust error handling, we ensure:
1.  **Stability:** The application continues running even if a specific browser operation fails.
2.  **Observability:** Failures are logged to the console, aiding remote debugging.
3.  **Graceful Degradation:** Features can fail gracefully without affecting the core user experience.

## 📂 Affected Files
The primary file requiring refactoring is:
- `src/components/roadmap/diagram.rs` (Specifically the `scroll_to_first_match` function)
- `src/components/ui/header.rs` (Scroll event handling)

## 🛠 Step-by-Step Implementation Guide

### Step 1: Refactor `scroll_to_first_match` in `src/components/roadmap/diagram.rs`

The current implementation uses multiple `match` statements and early returns, which is good, but the final `unwrap_or(0.0)` for `scroll_y()` can be improved, and the logic can be consolidated into a single `Result` chain for clarity.

**Action:** Modify `scroll_to_first_match` in `src/components/roadmap/diagram.rs`.

```rust
// src/components/roadmap/diagram.rs

// ... existing imports ...
use web_sys::ScrollBehavior; // Ensure this is imported if not already

/// Scroll to the first matching node in the viewport using web-sys (Pure Rust)
fn scroll_to_first_match(topic_id: &str) {
    // Define a local function to handle the Result chain
    let result: Result<(), JsValue> = (|| {
        let window = web_sys::window().ok_or_else(|| JsValue::from_str("Window not found"))?;
        let document = window.document().ok_or_else(|| JsValue::from_str("Document not found"))?;

        let selector = format!("[data-topic-id=\"{}\"]", topic_id);
        let element = document.query_selector(&selector)?
            .ok_or_else(|| JsValue::from_str(&format!("Element with selector {} not found", selector)))?;

        // Create ScrollToOptions with smooth behavior
        let scroll_options = web_sys::ScrollToOptions::new();
        scroll_options.set_behavior(ScrollBehavior::Smooth);

        // Get element position and scroll with offset
        let rect = element.get_bounding_client_rect();
        
        // Use `window.scroll_y()` which returns a Result<f64, JsValue>
        let current_scroll = window.scroll_y()?; 
        
        let target_y = current_scroll + rect.top() - 150.0; // 150px offset from top

        scroll_options.set_top(target_y);
        window.scroll_to_with_scroll_to_options(&scroll_options);

        Ok(())
    })(); // Execute the closure immediately

    // Log the error if the operation failed
    if let Err(e) = result {
        // Use the `log` crate for structured logging
        log::warn!("Failed to scroll to topic {}: {:?}", topic_id, e);
    }
}
```

### Step 2: Refactor Scroll Event Handling in `src/components/ui/header.rs`

The `create_effect` block in `Header` component uses `unwrap_or(0.0)` for `scroll_y()`. While this is a safe fallback, we should ensure the entire setup process is robust.

**Action:** Modify the `create_effect` block in `src/components/ui/header.rs`.

```rust
// src/components/ui/header.rs

use leptos::wasm_bindgen::JsCast;
use leptos::*;

#[component]
pub fn Header(search_term: ReadSignal<String>, on_search: Callback<String>) -> impl IntoView {
    let (is_scrolled, set_is_scrolled) = create_signal(false);

    // Handle scroll for sticky header effect
    create_effect(move |_| {
        // Check for window existence once at the start of the effect
        if let Some(window) = web_sys::window() {
            let closure = leptos::wasm_bindgen::closure::Closure::wrap(Box::new(move || {
                // Inside the closure, we handle the scroll_y() result explicitly
                if let Some(w) = web_sys::window() {
                    // Replace unwrap_or(0.0) with explicit Result handling
                    let scroll_y = w.scroll_y().unwrap_or_else(|e| {
                        // Log the error if getting scroll position fails
                        log::error!("Failed to get scroll_y: {:?}", e);
                        0.0 // Fallback to 0.0
                    });
                    
                    let scrolled = scroll_y > 20.0;
                    set_is_scrolled.set(scrolled);
                }
            }) as Box<dyn Fn()>);

            // The add_event_listener_with_callback returns a Result<(), JsValue>
            if let Err(e) = window.add_event_listener_with_callback("scroll", closure.as_ref().unchecked_ref()) {
                log::error!("Failed to add scroll event listener: {:?}", e);
            }

            on_cleanup(move || {
                if let Some(window) = web_sys::window() {
                    // The remove_event_listener_with_callback also returns a Result<(), JsValue>
                    if let Err(e) = window.remove_event_listener_with_callback(
                        "scroll",
                        closure.as_ref().unchecked_ref(),
                    ) {
                        log::error!("Failed to remove scroll event listener: {:?}", e);
                    }
                }
                // closure is dropped here automatically
            });
        } else {
            // Log if the window is not available (e.g., in a server-side context, though unlikely for CSR)
            log::warn!("Cannot set up scroll effect: web_sys::window() returned None.");
        }
    });

    // ... rest of the component remains the same ...
}
```

### Step 3: Ensure Logging is Configured

For the `log::warn!` and `log::error!` calls to work, the logging setup in `src/main.rs` must be correct. The existing code already includes:

```rust
// src/main.rs
use leptos::*;
use rust_roadmap::app::App;

fn main() {
    console_error_panic_hook::set_once();
    // Ensure console_log is initialized for the log calls to appear
    // The Cargo.toml already includes `console_log = "1"`.
    // Add the initialization line if it's missing:
    // console_log::init_with_level(log::Level::Warn).expect("couldn't initialize logging");
    
    mount_to_body(|| view! { <App/> })
}
```

**Action:** Verify that `console_log::init_with_level` is called in `src/main.rs` (or `src/lib.rs` if the app is a library). If it's not present, add it to ensure the error messages are visible in the browser console.

## ✅ Verification and Testing
1.  The application compiles successfully (`trunk build`).
2.  The application loads and the header scroll effect works as expected.
3.  Open the browser's Developer Console. If any of the browser API calls fail (e.g., if the environment is restricted), the application should **not** panic. Instead, a `log::warn!` or `log::error!` message should appear in the console, and the application should continue running.
4.  Verify that the `scroll_to_first_match` function still correctly scrolls the view when a search result is found.
