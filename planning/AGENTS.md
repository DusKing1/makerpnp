# PLANNING MODULE

**Parent:** [Root AGENTS.md](../AGENTS.md)

## OVERVIEW

Assembly planning — 10 crates implementing the Crux core/shell architecture for PCB assembly job management.

## STRUCTURE

```
planning/
├── planning/           # Pure domain logic (Project, Phase, Process, Placement) — NO Crux, NO I/O
├── planner_app/        # Crux Core — implements crux_core::App, 70+ Events, Model, Effects
├── planner_cli/        # Crux Shell (CLI) — crossbeam-channel effect loop, auto-save
├── planner_gui_egui/   # Crux Shell (GUI) — egui frontend (see planner_gui_egui/AGENTS.md)
├── variantbuilder_app/ # Crux Core — EDA file normalization (separate from planner)
├── variantbuilder_cli/ # Crux Shell (CLI) — DipTrace/KiCad/EasyEDAPro → normalized placements
├── stores/             # Data source abstraction — CSV/file loading, aggregates assembly+mappers
├── assembly/           # Assembly variant processing — filters placements by variant rules
├── part_mapper/        # Part mapping rules engine — matches parts via criteria
└── package_mapper/     # Package mapping rules engine — matches packages via criteria
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Add new Event | `planner_app/src/lib.rs` | Add variant to `Event` enum + handle in `update()` |
| Add CLI subcommand | `planner_cli/src/opts.rs` (719 lines) | Clap derive, map to Event in `main.rs` |
| Add project domain logic | `planning/src/project.rs` (2722 lines) | Pure logic — `Project` struct methods |
| Add new Phase/Process behavior | `planning/src/phase.rs`, `planning/src/process.rs` | State machine transitions |
| Add data source | `stores/src/` | Implement loader, register in stores aggregator |
| Add part/package mapping rule | `part_mapper/src/`, `package_mapper/src/` | Uses `criteria` crate for matching |
| Add EDA tool support | `variantbuilder_app/src/lib.rs` + `variantbuilder_cli/` | Add parser in `eda/`, wire through app |
| Placement sorting | `planning/src/placement.rs` | `PlacementSortingItem`, `PlacementSortingMode` |

## DEPENDENCY FLOW

```
Shells (planner_cli, planner_gui_egui, variantbuilder_cli)
  → Cores (planner_app, variantbuilder_app)
    → stores
      → assembly, part_mapper, package_mapper
        → planning (pure domain)
          → pnp, eda, common/*
```

No cycles. All dependencies flow downward.

## CONVENTIONS

- **Crux pattern**: Core is pure — all side effects via `Effect` enum, handled by Shell.
- **CLI auto-save**: Shell checks `view.project_modified` / `view.pcbs_modified` after every Render effect.
- **Sequence tests**: `planner_cli/tests/planner.rs` (4324 lines) — uses static mutex + sequence numbering for ordered test execution.
- **Test builders**: `tests/common/` — `TestProject`, `LoadOutCSVBuilder`, `PhasePlacementsCSVBuilder` etc.
- **ObjectPath**: Unified locator format `/pcb_instance/pcb_unit/ref_des` (e.g. `/1/2/R1`).
- **Process state machine**: Process → Operations → Tasks, three-level status tracking.
- **`testing` feature flag**: Dev-dependencies use `features = ["testing"]` to avoid polluting release builds.

## ANTI-PATTERNS

- Do NOT add I/O to `planning/` crate — it must stay pure domain logic.
- Do NOT handle `Effect::ProjectView` / `Effect::PcbView` in CLI shell — only GUI uses those.
- Do NOT start multiple phase tasks simultaneously — only one should be active.
