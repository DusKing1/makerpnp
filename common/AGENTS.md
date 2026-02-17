# COMMON UTILITIES

**Parent:** [Root AGENTS.md](../AGENTS.md)

## OVERVIEW

6 shared utility crates — foundational types and helpers used across all domain crates.

## STRUCTURE

```
common/
├── args/       # Primitive argument type (`Arg` enum: Boolean, String, Integer) for i18n
├── cli/        # CLI utilities: clap parsers, tracing setup, internal→CLI type adapters
├── criteria/   # Rule matching engine: `FieldCriterion` trait, ExactMatch + RegexMatch strategies
├── i18n/       # Fluent-based i18n: loads .ftl translations, wraps egui_i18n
├── math/       # Numeric utilities: angle, decimal, ratio, math operations
└── util/       # Core utilities: Source enum, sorting, path helpers, dynamic typing, assertions
```

## WHERE TO LOOK

| Task | Crate | Key File | Notes |
|------|-------|----------|-------|
| Add rule matching strategy | `criteria` | `src/lib.rs` | Implement `FieldCriterion` trait — used by `part_mapper` & `package_mapper` |
| Add CLI arg type adapter | `cli` | `src/args.rs` | Internal domain types → `*Arg` enums (clap ValueEnum) |
| Add CLI parser | `cli` | `src/parsers.rs` | Clap argument parsers |
| Configure tracing | `cli` | `src/tracing.rs` | Behind `tracing` feature flag |
| Add math operation | `math` | `src/ops.rs`, `src/angle.rs` | Angle/decimal/ratio math |
| Add file source type | `util` | `src/source.rs` | `Source` enum — file/URL abstraction |
| Add sorting helper | `util` | `src/sorting.rs` | `SortOrder` enum |
| Add dynamic typing | `util` | `src/dynamic/` | `AsAny` + `DynamicEq` traits — enables trait object equality |
| Add test utility | `util` | `src/test/` | Behind `testing` feature flag |
| Add i18n translation | `i18n` | `src/lib.rs` | Loads Fluent `.ftl` files, wraps `egui_i18n` |
| Add i18n argument type | `args` | `src/lib.rs` | `Arg` enum for Fluent argument conversion |

## CONVENTIONS

- **`testing` feature**: `util/src/test/` gated behind `#[cfg(any(test, feature = "testing"))]` — prevents test utilities from polluting release builds.
- **Dynamic equality**: `criteria` uses `AsAny` + `DynamicEq` from `util/dynamic/` for trait object equality (`dyn FieldCriterion`).
- **i18n feature flags**: `i18n` has `json` and `args` features for different Fluent argument conversion strategies.
- **Edition mix**: `i18n`, `math`, `args` use Rust edition 2024; `cli`, `criteria`, `util` use 2021.

## ANTI-PATTERNS

- Do NOT add domain-specific logic here — these crates are cross-cutting utilities only.
- Do NOT bypass `FieldCriterion` trait for rule matching — always implement the trait interface.
