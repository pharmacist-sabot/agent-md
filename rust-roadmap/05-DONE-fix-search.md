# Search State Architecture

## Overview
The Search State in the `RoadmapPage` component is responsible for filtering visible `RoadmapNode` elements based on user input. It implements a **Fine-Grained Reactive** pattern using Leptos `Signal`s to ensure that the rendering pipeline updates efficiently only when the search term changes.

## Type System & Ownership

In Rust, strictly managing ownership of the state is critical for memory safety and thread safety in concurrent environments (even conceptually in reactive systems).

### Signal Handles
The search state is instantiated using `create_signal`. This function returns a tuple of two distinct handles:

1.  **`ReadSignal<String>` (The Getter):**
    *   Ownership: Immutable access to the value.
    *   Purpose: To be passed down to child components (`RoadmapDiagram`) that need to *read* the state.
    *   Reactivity: Calling `.get()` on this handle establishes a reactive dependency tracking subscription for the current scope.

2.  **`WriteSignal<String>` (The Setter):**
    *   Ownership: Mutable access to the value.
    *   Purpose: To be used exclusively by the component that owns the state (`RoadmapPage`) or via `Callback`s passed to event listeners (Header).

### Initialization
We initialize the signal with an owned `String` because the user input is dynamic and heap-allocated.

```rust
// src/routes/roadmap.rs

let (search_term, set_search_term) = create_signal(String::new());
```

---

## Event Handling (The Controller)

We wrap the `WriteSignal` in a Leptos `Callback` to bridge the gap between DOM events and our internal Rust state. This decouples the DOM logic from the business logic.

```rust
let handle_search = Callback::new(move |val: String| {
    set_search_term.set(val);
});
```

*   **Ownership Transfer:** The `val` (String) is moved into the `set_search_term` closure.
*   **State Update:** `set_search_term.set(val)` triggers the reactive system, notifying all subscribers (readers) of the change.

---

## Reactive Propagation (The Architecture)

To make the search functional, we must pass the **`ReadSignal`** down the component tree, NOT the `String` value itself. Passing the `String` would create a snapshot of the value at render time, breaking the reactive link.

### 1. Passing the Signal (RoadmapPage -> RoadmapDiagram)

```rust
view! {
    <RoadmapDiagram
        // ...
        search_term=search_term // Passing the ReadSignal
        // ...
    />
}
```

### 2. Props Definition (Diagram)

The component props must explicitly accept the `ReadSignal`.

```rust
// src/components/roadmap/diagram.rs

#[derive(Clone)]
pub struct DiagramData {
    // ...
    pub search_term: ReadSignal<String>, // Strict Type: ReadSignal
    // ...
}
```

### 3. The Filter Logic (The "Active" Listener)

This is where the filtering happens. Inside the rendering loop, we call `.get()` on the signal.

**CRITICAL:** This `.get()` call is what creates the reactive subscription. Without it, the component does not know it needs to re-render when the user types.

```rust
// src/components/roadmap/diagram.rs

// Create the dependency by calling .get()
let term = props.search_term.get();
let lower_term = term.to_lowercase();

let node_props: Vec<_> = props.layout.topics.iter().filter_map(|tp| {
    let topic = find_topic(props.topics, tp.topic_id)?;

    // Filter Logic:
    // 1. If term is empty, return true (Show all)
    // 2. Else, check if topic title contains the term (Case-insensitive)
    if !term.is_empty() && !topic.title.to_lowercase().contains(&lower_term) {
        return None; // Discard this node
    }

    Some(NodeData { /* ... */ })
}).collect();
```

---

## Common Pitfalls

### The "Zombie State" (Disconnected State)

**The Problem:**
A common mistake in Leptos (and reactive systems) is defining the state but failing to connect it to the view logic.

```rust
// ❌ INCORRECT
// State exists, but props.search_term is never accessed
let node_props = props.layout.topics.iter()...map(|tp| {
    // Logic here does NOT call props.search_term.get()
}).collect();
```
*   **Result:** The state updates when typing, but the UI never re-renders because no subscription was created.

**The Solution:**
Always ensure the `ReadSignal` is accessed inside the reactive scope (the view macro or the deriving closure).

```rust
// ✅ CORRECT
// State is accessed, creating a subscription
let term = props.search_term.get(); 
// Proceed with logic...
```

### String Allocation Optimization
In a strict high-performance scenario, one might consider `Cow<'_, str>` to avoid allocation when the search term is empty. However, given that user input via `input` events is already a `String`, using `String` for the signal is the idiomatic and memory-safe choice here.

## Summary
*   **State Source:** `create_signal(String::new())` in `RoadmapPage`.
*   **Data Flow:** `WriteSignal` (Header) -> `Update` -> `ReadSignal` (Diagram) -> `Filter` -> `Re-render`.
*   **Reactivity Key:** The `.get()` call inside the iterator closure is the heartbeat of the search feature.