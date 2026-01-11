# Task: Add Rust Unit Tests to `warfarin_logic/src/lib.rs`

## Objective
Establish a robust test suite for the core calculation logic to ensure reliability and prevent regressions. We aim for **Test Coverage** on the `generate_suggestions_rust` function and its internal logic.

**Constraints:**
1.  **Do NOT modify existing logic:** Only add code. Do not change function signatures or algorithms unless they are buggy (if found, flag them instead of fixing blindly).
2.  **Use Standard Library:** Leverage `cargo test`.
3.  **Integration & Unit:** Test the public API (`generate_suggestions_rust`) effectively. Test internal helpers (`find_comb`) only if they are exposed or easily reachable via the public API.

---

## Implementation Guide

### 1. Create Test Module
Add the following module structure at the **bottom** of `warfarin_logic/src/lib.rs`.

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use wasm_bindgen::JsValue;

    // Helper to create a valid JS Input for the WASM function
    fn create_input(
        dose: f64,
        allow_half: bool,
        pills: Vec<u8>,
        pattern: &str,
    ) -> JsValue {
        // Create JSON manually or use serde_json if available in dev-dependencies
        // Ideally, use the struct and serde_wasm_bindgen::to_value
        let input = CalculationInput {
            weekly_dose: dose,
            allow_half: allow_half,
            available_pills: pills,
            special_day_pattern: match pattern {
                "fri-sun" => SpecialDayPattern::FriSun,
                _ => SpecialDayPattern::MonWedFri,
            },
            days_until_appointment: 7,
            start_day_of_week: 0,
        };
        serde_wasm_bindgen::to_value(&input).unwrap()
    }

    // Helper to assert results are not empty and dose is correct
    fn assert_valid_result(output: JsValue, expected_dose: f64) {
        let results: Vec<FinalOutput> = serde_wasm_bindgen::from_value(output).unwrap();
        assert!(!results.is_empty(), "Should find at least one solution");
        // Check the first result's dose accuracy (tolerance 0.1)
        let diff = (results[0].weekly_dose_actual - expected_dose).abs();
        assert!(diff < 0.1, "Dose calculation mismatch");
    }
}
```

### 2. Required Test Cases
The AI must implement at least **15 test cases** covering the following scenarios:

#### A. Happy Path (Standard Calculations)
1.  **Simple Uniform Dose:** `weekly_dose: 21.0`, `pills: [1, 2, 3, 5]`.
    *   *Expect:* Uniform solution of 3mg per day (3 x 7 = 21).
2.  **Uniform with 5mg:** `weekly_dose: 35.0`, `pills: [1, 5]`.
    *   *Expect:* 5mg every day.
3.  **Mixed Pills Uniform:** `weekly_dose: 14.0`, `pills: [2, 3]`.
    *   *Expect:* 2mg/day (Uniform).
4.  **Complex Combination:** `weekly_dose: 17.5`, `pills: [1, 2, 3, 5]`, `allow_half: true`.
    *   *Expect:* A valid solution using half-pills (e.g., 2.5mg/day).

#### B. Half-Pill Constraints
5.  **Half Pills Allowed:** `weekly_dose: 10.5`, `allow_half: true`.
    *   *Expect:* Solutions containing `is_half: true` pills.
6.  **Half Pills Disallowed:** `weekly_dose: 10.5`, `allow_half: false`.
    *   *Expect:* Solutions **MUST NOT** contain half-pills. If impossible, return empty or closest integer match depending on business logic.
7.  **Rounding Logic:** `weekly_dose: 10.8`, `allow_half: true`.
    *   *Expect:* Should handle floating point correctly (likely match 10.5 or 11.0).

#### C. Limited Pill Availability (Edge Cases)
8.  **Only 1mg Pills:** `weekly_dose: 7.0`, `pills: [1]`.
    *   *Expect:* 1mg x 7 every day.
9.  **Only 5mg Pills (Impossible):** `weekly_dose: 3.0`, `pills: [5]`.
    *   *Expect:* **Zero results** (cannot make 3mg with 5mg pills).
10. **Specific Combinations:** `weekly_dose: 28.0`, `pills: [1, 5]`.
    *   *Expect:* Check if it can solve 4mg/day using only 1s and 5s (4x1mg).

#### D. Non-Uniform Patterns (Special Days)
11. **Skip Days (Stop Days):** Enable `num_stop_days > 0` logic.
    *   *Expect:* Verify that the output results contain schedules where `is_stop_day` is true for specific days.
12. **Special Day Pattern:** `weekly_dose: 35.0`, `pattern: "fri-sun"`.
    *   *Expect:* Verify results handle Fri/Sun specific doses correctly.

#### E. Boundary & Negative Cases
13. **Zero Dose:** `weekly_dose: 0.0`.
    *   *Expect:* Should return empty vector or handle gracefully (no crash).
14. **Negative Dose:** `weekly_dose: -5.0`.
    *   *Expect:* Should return empty vector.
15. **Unrealistically High Dose:** `weekly_dose: 200.0`.
    *   *Expect:* Should not panic or hang (Performance check).
16. **Empty Input (No Pills):** `weekly_dose: 10.0`, `pills: []`.
    *   *Expect:* Should return empty vector (Error message in real app, empty array in logic).

#### F. Output Structure Validation
17. **Data Integrity:** Run a standard calculation and inspect the JSON output.
    *   *Assert:* Ensure `weekly_schedule` contains exactly 7 days.
    *   *Assert:* Ensure `DaySchedule` indices are 0-6.
    *   *Assert:* Ensure `total_dose` in each day aligns with the `pills` list in the data structure.

---

## Validation Steps

After generating the code:

1.  **Run Tests:** Execute `cargo test` inside `warfarin_logic/`.
2.  **Verify Output:** Ensure all 15+ tests pass.
3.  **Check Timing:** Ensure tests complete in under 2-3 seconds total. If a test hangs, the recursive logic might have an infinite loop edge case.
4.  **Check Panic:** Ensure no test triggers a `panic!` (use `assert!` instead).

## Code Quality Standards for Tests
*   **Naming:** Tests must be descriptive (e.g., `test_uniform_dose_simple_5mg_pills`).
*   **Independence:** Tests must not rely on the state of previous tests.
*   **Formatting:** Use `cargo fmt` to ensure consistency.