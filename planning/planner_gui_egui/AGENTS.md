# PLANNER GUI (egui)

**Parent:** [Planning AGENTS.md](../AGENTS.md)

## OVERVIEW

egui/eframe GUI shell for the Planner Crux core — 65 source files, largest frontend in the project.

## STRUCTURE

```
planner_gui_egui/src/
├── main.rs                # Entry point — eframe::run_native, i18n init, profiling setup
├── lib.rs                 # Module declarations
├── ui_app.rs (940 lines)  # Root app state — manages projects, pcbs, tabs, toolbar
├── planner_app_core.rs    # PlannerCoreService — wraps Crux Core, Effect → PlannerAction
├── config.rs              # Persistent app configuration (JSON)
├── command.rs             # Command pattern for UI actions
├── ui_app/
│   └── app_tabs/          # Top-level tab types (home, new_project, new_pcb, project, pcb)
├── project/
│   ├── mod.rs (2995 lines)# Project view — largest file, orchestrates all project tabs
│   ├── toolbar.rs         # Project-specific toolbar
│   ├── process.rs         # Process management UI
│   ├── tabs/              # 9 tabs: overview, explorer, pcb, unit_assignments, parts,
│   │                      #   load_out, placements, phase, process, issues
│   ├── tables/            # Data tables: placements, parts, load_out
│   └── dialogs/           # Dialogs: add_phase, placement_orderings, package_sources, errors
├── pcb/
│   ├── mod.rs (1190 lines)# PCB editor view
│   └── tabs/              # panel, configuration, gerber_viewer, explorer
├── ui_components/         # Reusable UI components (gerber_viewer_ui)
├── widgets/               # Custom widgets (list_box, augmented_list_selector)
├── dialogs/               # App-level dialogs (manage_gerbers)
├── forms/                 # Form validation utilities
├── filter/                # Data filtering logic
├── i18n/                  # GUI-specific i18n conversions
├── runtime/               # Async runtime (tokio_runtime, legacy_runtime)
├── task/                  # Background task management
├── fonts.rs               # Font configuration
├── profiling.rs           # puffin profiling integration
├── tabs.rs                # Tab trait and egui_dock integration
├── toolbar.rs             # Shared toolbar components
├── file_picker.rs         # File dialog integration (rfd)
└── ui_util.rs             # UI helper functions
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Add project tab | `project/tabs/` | Create `*_tab.rs`, register in `project/mod.rs` |
| Add PCB editor tab | `pcb/tabs/` | Create `*_tab.rs`, register in `pcb/mod.rs` |
| Add data table | `project/tables/` | Uses `egui_deferred_table` (forked) |
| Add dialog | `project/dialogs/` or `dialogs/` | Project-scoped vs app-scoped |
| Add toolbar button | `project/toolbar.rs` or `toolbar.rs` | Project vs app level |
| Modify Crux integration | `planner_app_core.rs` | Effect → PlannerAction mapping |
| Add widget | `widgets/` | Reusable across views |
| Change app layout | `ui_app.rs` | egui_dock tab management |
| Add i18n key | `i18n/conversions.rs` + `.ftl` files | Fluent format |
| Background async work | `runtime/`, `task/` | tokio-based |

## CONVENTIONS

- **Effect handling**: `PlannerCoreService.update(event)` returns `Vec<PlannerAction>` — caller processes actions sequentially.
- **Tab system**: Uses `egui_dock` for dockable/floatable tabs. Each tab implements a trait.
- **Large view files**: `project/mod.rs` (2995 lines) and `pcb/mod.rs` (1190 lines) orchestrate their respective views — read these first when understanding UI flow.
- **Edition 2024**: This crate uses Rust edition 2024 (newer than most crates in workspace).
- **Assets required**: Must run from crate directory — logos and i18n `.ftl` files loaded via relative paths.

## ANTI-PATTERNS

- Do NOT float tabs containing data tables in debug mode — causes panic (egui_dock #278).
- Do NOT add business logic here — belongs in `planner_app` (Crux Core) or `planning` (domain).
- Do NOT bypass `PlannerCoreService` — all state changes go through Crux Events.
