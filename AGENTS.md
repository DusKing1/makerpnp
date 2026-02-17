# PROJECT KNOWLEDGE BASE

**Generated:** 2026-02-17
**Commit:** 475c468
**Branch:** master

## OVERVIEW

MakerPnP — cross-platform Pick-and-Place machine software for Makers. Pure Rust Cargo workspace (31 crates), using Crux core/shell architecture with egui GUI and CLI frontends.

## STRUCTURE

```
makerpnp/
├── common/           # 6 shared utility crates (see common/AGENTS.md)
├── eda/              # EDA tool integration (eda, eda_units) — DipTrace, KiCad, EasyEDAPro
├── gerber/           # Gerber file handling (gerber, gerber_viewer_egui)
├── pnp/              # Core PnP types (Part, Placement, Package, ObjectPath, LoadOut)
├── planning/         # Assembly planning — 10 crates, largest module (see planning/AGENTS.md)
├── assets/           # Logos, screenshots
├── .github/          # CI workflow (rust.yml)
├── Hacks.md          # Documented hack patterns with identifiers
└── KnownIssues.md    # Known limitations and workarounds
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Add CLI subcommand | `planning/planner_cli/src/opts.rs` | Uses clap 4.5, add ValueEnum adapters in `common/cli/src/args.rs` |
| Add GUI tab/dialog | `planning/planner_gui_egui/src/` | See `planning/planner_gui_egui/AGENTS.md` |
| Add Crux Event | `planning/planner_app/src/lib.rs` | 70+ events in `Event` enum, handle in `update()` |
| Add EDA tool support | `eda/eda/src/` | Implement parser, add variant to `variantbuilder_cli` |
| Part/Placement types | `pnp/pnp/src/` | Core domain types shared across all crates |
| Planning domain logic | `planning/planning/src/` | Pure domain — no Crux, no I/O |
| Data loading (CSV/file) | `planning/stores/src/` | Aggregates assembly, part_mapper, package_mapper |
| Rule matching | `common/criteria/src/lib.rs` | `FieldCriterion` trait — exact/regex strategies |
| i18n translations | `common/i18n/` + GUI `.ftl` files | Based on egui-i18n + Fluent |
| Gerber rendering | `gerber/` | Uses forked gerber_viewer library |

## ARCHITECTURE

### Crux Core/Shell Pattern

```
User Input → Event → Core.update() → Effect → Shell → Render
                ↑                                  ↓
                └──────── New Event ───────────────┘
```

- **Core** (`planner_app`, `variantbuilder_app`): Implements `crux_core::App`. Pure functions, no I/O.
- **Shell CLI** (`planner_cli`, `variantbuilder_cli`): `crossbeam-channel` effect loop. Auto-saves on modify.
- **Shell GUI** (`planner_gui_egui`): `PlannerCoreService` wraps Core, converts Effect → PlannerAction for egui.

### Dependency Flow (top → bottom, no cycles)

```
Shells (cli, gui)
  → Crux Cores (planner_app, variantbuilder_app)
    → Domain Support (stores, assembly, part_mapper, package_mapper)
      → Domain Core (planning — pure logic, no I/O)
        → Shared Types (pnp, eda, gerber, common/*)
```

### 4 Executables

| Binary | Crate | Purpose |
|--------|-------|---------|
| `planner_gui_egui` | `planning/planner_gui_egui` | GUI planner (egui + eframe) |
| `planner_cli` | `planning/planner_cli` | CLI planner (same Crux core) |
| `variantbuilder_cli` | `planning/variantbuilder_cli` | EDA → normalized placements |
| `gerber_viewer_egui` | `gerber/gerber_viewer_egui` | Standalone Gerber viewer |

## CONVENTIONS

- **Formatting**: `cargo +nightly fmt` — max_width=120, chain_width=40, struct_lit_single_line=false, group_imports=StdExternalCrate
- **Workspace deps**: ALL versions centralized in root `Cargo.toml` `[workspace.dependencies]`. Crates reference via `{ workspace = true }`.
- **Fork deps**: egui, egui_taffy, egui_ltreeview, egui_deferred_table, gerber_viewer — all forked under MakerPnP GitHub org with specific revs.
- **Resolver v2**: Required due to `testing` feature flag on dev-dependencies.
- **CLI adapters**: Internal types → `*Arg` enums in `common/cli/src/args.rs` via `From/Into`.
- **Test frameworks**: `rstest` (parameterized), `assert_cmd` + `assert_fs` (CLI integration).
- **Test data builders**: Builder pattern in `tests/common/` — `TestProject`, CSV builders.
- **Hack documentation**: Document hacks in `Hacks.md` with identifier comment (e.g. `HACK: table-resize-hack`), not scattered TODOs.

## ANTI-PATTERNS (THIS PROJECT)

- **No `as any` / `@ts-ignore` equivalent**: Do not suppress type errors.
- **No scattered TODO/FIXME**: Use `Hacks.md` and `KnownIssues.md` for tracking.
- **GUI must run from source dir**: GUI executables require relative asset paths — run from crate directory.
- **Don't float data-table tabs in debug mode**: Causes panic (egui_dock bug #278).
- **Single task active**: Only one phase task should be started/incomplete at a time.
- **PCB changes require project reload**: Save project → close → edit PCB → reopen.

## COMMANDS

```bash
# Build
cargo build --release

# Test
cargo test --verbose

# Format (requires nightly)
cargo +nightly fmt

# Run GUI (from crate dir)
cd planning/planner_gui_egui && ../../target/release/planner_gui_egui

# Run CLI
./target/release/planner_cli --help
./target/release/variantbuilder_cli --help
```

## NOTES

- **CI**: GitHub Actions on push/PR to master. Ubuntu. Needs `libdbus-1-dev`.
- **Edition mix**: Most crates use 2021, newer GUI crates use 2024.
- **Gerber crates tightly coupled**: gerber_viewer, gerber-types, gerber_parser — ensure only one of each in dependency tree.
- **chrono over time**: Project chose chrono (13k dependents) over time crate (3k dependents).
- **License**: TBD (likely GPL3, Apache, or MIT).
