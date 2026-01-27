# Unit Testing Implementation: Layout Engine Robustness

## 🎯 Goal
Implement a comprehensive suite of unit tests within `src/layout/tree.rs` to verify the correctness and determinism of the `compute_layout` function. This is crucial for ensuring that the "Fishbone Layout Engine" maintains visual fidelity under various data structures and scales.

## 💡 Motivation
The layout engine is the core architectural component responsible for the visual representation of the roadmap. Any regression in this logic can lead to a broken or visually incorrect diagram. By achieving high test coverage on `compute_layout`, we guarantee:

1.  **Determinism:** The same input data always yields the exact same coordinates.
2.  **Robustness:** The engine handles edge cases like empty sections, mixed layouts (List/Grid), and large topic counts correctly.
3.  **Regression Prevention:** Future refactoring or feature additions will not inadvertently break the core layout logic.

## 📂 Affected File
- `src/layout/tree.rs`

## 🛠 Step-by-Step Implementation Guide

### Step 1: Setup the Test Module

In `src/layout/tree.rs`, add the following test module block at the end of the file. We need to import the necessary domain models from `crate::models::roadmap` and a crate for reliable floating-point comparison.

**Action:** Add the following block to the end of `src/layout/tree.rs`.

```rust
// src/layout/tree.rs (at the end of the file)

#[cfg(test)]
mod tests {
    use super::*;
    use crate::models::roadmap::{Dependency, Level, Placement, Section, SectionLayout, Topic, TopicType};
    // Note: You may need to add 'float_cmp = "0.9"' to Cargo.toml dependencies for this to work.
    // If not possible, use a simple epsilon comparison (e.g., (a - b).abs() < 0.001).
    // For this guide, we assume a simple epsilon comparison is used.

    // Helper function to create a mock topic
    fn mock_topic(id: &'static str, section_id: &'static str, placement: Placement) -> Topic {
        Topic {
            id,
            title: id,
            section_id,
            level: Level::Beginner,
            topic_type: TopicType::Sub,
            placement,
            row: None,
        }
    }

    // Helper function to create a mock section
    fn mock_section(id: &'static str, order: u8, layout: SectionLayout) -> Section {
        Section {
            id,
            title: id,
            order,
            layout,
        }
    }

    // Standard configuration for testing
    fn test_config() -> LayoutConfig {
        LayoutConfig::default()
    }

    // Epsilon for floating point comparison
    const EPSILON: f64 = 0.001;

    // ... Test functions will go here ...
}
```

### Step 2: Test Case 1: Simple Spine (Center Placement)

Verify the vertical stacking and center alignment of topics placed in the `Center`.

```rust
    #[test]
    fn test_simple_spine_layout() {
        let config = test_config();
        let sections = vec![
            mock_section("s1", 1, SectionLayout::List),
            mock_section("s2", 2, SectionLayout::List),
        ];
        let topics = vec![
            mock_topic("t1", "s1", Placement::Center),
            mock_topic("t2", "s2", Placement::Center),
        ];
        let dependencies = vec![];

        let result = compute_layout(&sections, &topics, &dependencies, &config);

        // t1 should be at the top of the layout
        let t1_pos = result.topics.iter().find(|p| p.topic_id == "t1").unwrap();
        assert!((t1_pos.x - (config.center_x - config.node_width / 2.0)).abs() < EPSILON);
        assert!((t1_pos.y - 100.0).abs() < EPSILON); // Initial Y offset

        // t2 should be below t1, separated by node_height + node_spacing_y + section_spacing
        let t2_pos = result.topics.iter().find(|p| p.topic_id == "t2").unwrap();
        let expected_y_t2 = t1_pos.y + config.node_height + config.node_spacing_y + config.section_spacing;
        assert!((t2_pos.x - (config.center_x - config.node_width / 2.0)).abs() < EPSILON);
        assert!((t2_pos.y - expected_y_t2).abs() < EPSILON);
    }
```

### Step 3: Test Case 2: Left and Right Branching (List Layout)

Verify that topics with `Left` and `Right` placement are correctly positioned relative to the center axis.

```rust
    #[test]
    fn test_left_right_branching() {
        let config = test_config();
        let sections = vec![mock_section("s1", 1, SectionLayout::List)];
        let topics = vec![
            mock_topic("t_main", "s1", Placement::Center),
            mock_topic("t_left", "s1", Placement::Left),
            mock_topic("t_right", "s1", Placement::Right),
        ];
        let dependencies = vec![];

        let result = compute_layout(&sections, &topics, &dependencies, &config);

        let t_main_pos = result.topics.iter().find(|p| p.topic_id == "t_main").unwrap();
        let t_left_pos = result.topics.iter().find(|p| p.topic_id == "t_left").unwrap();
        let t_right_pos = result.topics.iter().find(|p| p|p.topic_id == "t_right").unwrap();

        // Center Topic
        assert!((t_main_pos.x - (config.center_x - config.node_width / 2.0)).abs() < EPSILON);

        // Left Topic: Should be centered at (center_x - col_spacing)
        let expected_x_left = config.center_x - config.col_spacing - config.node_width / 2.0;
        assert!((t_left_pos.x - expected_x_left).abs() < EPSILON);
        // Should be vertically aligned with the next available row in the section
        let expected_y_left = t_main_pos.y + config.node_height + config.node_spacing_y;
        assert!((t_left_pos.y - expected_y_left).abs() < EPSILON);

        // Right Topic: Should be centered at (center_x + col_spacing)
        let expected_x_right = config.center_x + config.col_spacing - config.node_width / 2.0;
        assert!((t_right_pos.x - expected_x_right).abs() < EPSILON);
        // Should be vertically aligned with the next available row in the section
        let expected_y_right = t_main_pos.y + config.node_height + config.node_spacing_y;
        assert!((t_right_pos.y - expected_y_right).abs() < EPSILON);
    }
```

### Step 4: Test Case 3: Grid Layout Verification

Verify that the `SectionLayout::Grid { cols: 2 }` correctly places nodes horizontally within the section's column space.

```rust
    #[test]
    fn test_grid_layout() {
        let config = test_config();
        let sections = vec![mock_section("s1", 1, SectionLayout::Grid { cols: 2 })];
        let topics = vec![
            mock_topic("t_main", "s1", Placement::Center),
            mock_topic("t_l1", "s1", Placement::Left),
            mock_topic("t_l2", "s1", Placement::Left),
            mock_topic("t_r1", "s1", Placement::Right),
            mock_topic("t_r2", "s1", Placement::Right),
        ];
        let dependencies = vec![];

        let result = compute_layout(&sections, &topics, &dependencies, &config);

        let t_main_pos = result.topics.iter().find(|p| p.topic_id == "t_main").unwrap();
        let t_l1_pos = result.topics.iter().find(|p| p.topic_id == "t_l1").unwrap();
        let t_l2_pos = result.topics.iter().find(|p| p.topic_id == "t_l2").unwrap();

        // Left Column 1 (t_l1)
        let expected_x_l1 = config.center_x - config.col_spacing - config.node_width;
        let expected_y_l1 = t_main_pos.y + config.node_height + config.node_spacing_y;
        assert!((t_l1_pos.x - expected_x_l1).abs() < EPSILON);
        assert!((t_l1_pos.y - expected_y_l1).abs() < EPSILON);

        // Left Column 2 (t_l2) - Should be next to t_l1 horizontally
        let expected_x_l2 = expected_x_l1 + config.node_width + config.grid_col_spacing;
        assert!((t_l2_pos.x - expected_x_l2).abs() < EPSILON);
        assert!((t_l2_pos.y - expected_y_l1).abs() < EPSILON); // Same row
    }
```

### Step 4: Test Case 4: Total Dimensions and Edge Cases

Verify that the overall dimensions (`total_width`, `total_height`) are calculated correctly and that the engine handles empty data gracefully.

```rust
    #[test]
    fn test_total_dimensions_and_empty_data() {
        let config = test_config();
        let sections = vec![
            mock_section("s1", 1, SectionLayout::List),
            mock_section("s2", 2, SectionLayout::List),
        ];
        let topics = vec![
            mock_topic("t_left", "s1", Placement::Left),
            mock_topic("t_right", "s2", Placement::Right),
        ];
        let dependencies = vec![];

        let result = compute_layout(&sections, &topics, &dependencies, &config);

        // Total Width: Should span from the leftmost node to the rightmost node.
        // Leftmost X: config.center_x - config.col_spacing - config.node_width / 2.0
        // Rightmost X: config.center_x + config.col_spacing + config.node_width / 2.0
        let expected_width = (config.center_x + config.col_spacing + config.node_width / 2.0) - result.min_x;
        assert!((result.total_width - expected_width).abs() < EPSILON);

        // Total Height: Should be based on the lowest node (t_right) + final offset (100.0)
        let t_right_pos = result.topics.iter().find(|p| p.topic_id == "t_right").unwrap();
        let expected_height = t_right_pos.y + config.node_height + 100.0;
        assert!((result.total_height - expected_height).abs() < EPSILON);

        // Empty Data Test
        let empty_result = compute_layout(&[], &[], &[], &config);
        assert!((empty_result.total_width - 0.0).abs() < EPSILON);
        assert!((empty_result.total_height - 100.0).abs() < EPSILON); // Base height
    }
```

## ✅ Verification and Testing
After implementing the tests:

1.  Run the tests using `cargo test`.
2.  Ensure all tests pass.
3.  If a test fails, use the expected coordinate values to debug the logic within `compute_layout`.
4.  **Crucially:** The tests must be run with the same `LayoutConfig::default()` values to ensure determinism.
