# VectorCraft — custom branch notes

Local fork notes for the `custom` branch. They live here, not in `CLAUDE.md`, so syncing `main`
with upstream never conflicts on the agent instructions. Read `CLAUDE.md` first: its rules apply.

## Branches
| Branch | Use |
|---|---|
| `main` | Mirror of `upstream/main` (`storytold/vectorcraft`). Never customised; sync with `git fetch upstream && git merge --ff-only upstream/main`. |
| `custom` | Our changes. Bring upstream in with `git merge main` (or rebase) after syncing `main`. |

Remotes: `origin` = `Krstic-c/vectorcraft` (fork), `upstream` = `storytold/vectorcraft`.

## Changes on this branch
| Change | Where | Notes |
|---|---|---|
| Drag the icon column's left edge wider (≥ 110 pt) to show each panel's name beside its icon | `crates/ui-egui/src/dock.rs` (`column_width`, `panel_icon`) | Max 240 pt; a drag ending within 14 pt of the icons snaps back to icons only |
| Drag the docked Tools panel's right edge wider (≥ 110 pt): one row per tool with its name | `crates/ui-egui/src/toolbar.rs` (`width`, `body`) | Max 260 pt; forces a single column while wide; floating toolbar unchanged |
| State: `UiState.icon_column_width`, `UiState.toolbar_width` | `crates/ui-egui/src/state.rs` | Saved with preferences, `0.0` = icons only. Deliberately not in `Workspace` snapshots |
| Shared widgets `paint_icon_row`, `width_grip`, `LABELS_MIN` | `crates/ui-egui/src/widgets.rs` | |

Names come from the existing i18n catalog (`tl!`), so they follow the interface language
(`crates/ui-egui/src/i18n/zh-hans.tsv` for Simplified Chinese).

## Build environment (Windows)
- Rust (stable, ≥ 1.95) in `D:\Dev\rust`: user env `RUSTUP_HOME=D:\Dev\rust\rustup`, `CARGO_HOME=D:\Dev\rust\cargo`; `D:\Dev\rust\cargo\bin` on the user PATH.
- MSVC Build Tools in `D:\Dev\BuildTools` (C++ workload). The Windows SDK installs under `C:\Program Files (x86)\Windows Kits\10` regardless.
- `vswhere` doesn't see that install, so load its environment before cargo:

```bat
call "D:\Dev\BuildTools\VC\Auxiliary\Build\vcvars64.bat"
cd /d D:\Work\GitHub\vectorcraft
cargo test -p vectorcraft-ui-egui          :: 885+ tests, a few seconds once built
cargo clippy -p vectorcraft-ui-egui --all-targets -- -D warnings
cargo fmt -p vectorcraft-ui-egui -- --check
cargo run -p vectorcraft -- --control 7979 :: the app, with the JSON control channel
```
- `target/` is inside the repo (D: drive, gitignored). Expect 10+ GB after full builds.
- The first build downloads ~250 crates straight from crates.io (reachable without a mirror from this network).

## Known pitfalls
- `widget_rects` based tests find buttons by size; a wide (named) row doesn't match a 30×30 / 36×29 filter, so those tests keep the default width.
- A `ui.horizontal` row registers a second rect a few points wider than its button: de-duplicate by `top()` in tests.
- Passing `ui` and `ui.id()...` to a free function in one call trips the borrow checker: compute the id first.

## To do
- Look at the result in the running app (`ui.screenshot` over the control channel) and tune row heights/fonts.
- The bottom controls of a wide Tools panel (fill/stroke, draw mode) are still centred at their old size.
- Consider a menu entry / command for the widths (`window.iconNames`) so the control channel and MCP can set them.
