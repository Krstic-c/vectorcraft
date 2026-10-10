# VectorCraft — custom branch notes

Local fork notes for the `custom` branch. They live here, not in `CLAUDE.md`, so syncing `main`
with upstream never conflicts on the agent instructions. Read `CLAUDE.md` first: its rules apply.

## Branches
| Branch | Use |
|---|---|
| `main` | Mirror of `upstream/main` (`storytold/vectorcraft`). Never customised. |
| `custom` | Our changes, and the fork's **default branch** on GitHub. |

The fork (`origin`) has only these two branches; the ~235 branches it inherited from upstream were
deleted (they all still exist on `upstream`). Remotes: `origin` = `Krstic-c/vectorcraft` (fork),
`upstream` = `storytold/vectorcraft`.

## Syncing with upstream
Don't use GitHub's **Sync fork** button: the default branch (`custom`) has diverged from upstream,
so it only offers to discard our commits or merge blindly. Sync from a terminal:

```sh
git fetch upstream
git checkout main
git merge --ff-only upstream/main      # main is a pure mirror: this must fast-forward
git push origin main
git checkout custom
git merge main                         # resolve conflicts here (dock.rs, toolbar.rs, state.rs, widgets.rs)
cargo test -p vectorcraft-ui-egui      # see Build environment; then push
git push origin custom
```

If `--ff-only` refuses, `main` has a commit of ours on it: move that commit to `custom` before
syncing. Other upstream branches (feature work, PR branches) are fetched into `upstream/*`; none
are kept on `origin`.

## Releasing a build (Windows x64)
Output goes to `dist/release/` (gitignored): an MSI, a portable zip and `SHA256SUMS.txt`. The
release published so far is [`v0.8.0-custom.1`](https://github.com/Krstic-c/vectorcraft/releases/tag/v0.8.0-custom.1).

```powershell
# 1. Release build, as packaging/windows/package.ps1 does it (static CRT, locked deps). In a shell with
#    vcvars64.bat loaded (see Build environment).
$env:CARGO_TARGET_X86_64_PC_WINDOWS_MSVC_RUSTFLAGS = "-C target-feature=+crt-static"
$env:VECTORCRAFT_REQUIRE_WINRES = "1"
cargo build --release --locked -p vectorcraft -p vectorcraft-cli --target x86_64-pc-windows-msvc

# 2. MSI (+ the script's own portable zip) from the binaries just built; WiX lives on D:
$env:DOTNET_ROOT = "D:\Dev\dotnet"; $env:NUGET_PACKAGES = "D:\Dev\dotnet\nuget"
$env:PATH = "D:\Dev\dotnet;D:\Dev\dotnet\tools;$env:PATH"
$env:VECTORCRAFT_VERSION = "0.8.0-custom"      # names the files; the MSI's own version stays 0.8.0
$env:DIST = "D:\Dev\tmp\msi-out"
powershell -NoProfile -File packaging\windows\package.ps1 -Arch x64 -SkipBuild

# 3. Publish (the fork's Releases page; the tag is created on the pushed custom branch)
gh release create v0.8.0-custom.2 <msi> <zip> SHA256SUMS.txt --repo Krstic-c/vectorcraft --target custom --title "..." --notes-file <notes.md>
```

Our portable zip is hand-made (`vectorcraft.exe`, `vectorcraft-cli.exe`, README, licences, NOTICE,
`CUSTOM.md`): the script's one leaves out NOTICE. Check both exes before packaging: x64, GUI
subsystem 2 for `vectorcraft.exe`, console 3 for the CLI (the script does it, step 2).
Builds are unsigned (no certificate) and don't embed craft-fonts, so CJK text uses system fonts.

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
- .NET 8 SDK (zip install, no installer) in `D:\Dev\dotnet`, with WiX v5.0.2 as a tool in `D:\Dev\dotnet\tools` and its NuGet cache in `D:\Dev\dotnet\nuget`: only needed for the MSI. Set `DOTNET_ROOT`, `NUGET_PACKAGES` and PATH per shell (see Releasing); WiX's extension cache (~2 MB) still lands in `C:\Users\<you>\.wix`.
- `target/` is inside the repo (D: drive, gitignored). Expect 10+ GB after full builds; the release build for `x86_64-pc-windows-msvc` adds its own `target\x86_64-pc-windows-msvc\release`.
- The first build downloads ~250 crates straight from crates.io (reachable without a mirror from this network).

## Build environment (macOS)
- Rust via Homebrew (`brew install rust`); satisfies `rust-version = "1.95"` in `Cargo.toml` (gives 1.99). No rustup, no toolchain file in the repo, so this just works. Xcode Command Line Tools (`xcode-select -p`) must already be installed — needed for the linker.

```sh
cd ~/Documents/GitHub/vectorcraft
cargo test -p vectorcraft-ui-egui          # 885 tests, ~4s once built (clean build pulls ~250 crates, several minutes)
cargo clippy -p vectorcraft-ui-egui --all-targets -- -D warnings
cargo fmt -p vectorcraft-ui-egui -- --check
VECTORCRAFT_NO_PREFS=1 cargo run -p vectorcraft -- --control 7979   # see Known pitfalls: always set this for a test run
```
- Verified 2026-10-10: all three checks clean, and `ui.drag` over the control channel both collapses (`icon_column_width` → `0.0`) and restores (≥ `LABELS_MIN`) the icon column live in the running app — matches the Windows-side behaviour.
- Without `VECTORCRAFT_NO_PREFS=1`, a test run reads/writes the real `~/Library/Application Support/VectorCraft/` (`ui.json`, `Data Recovery/`, `logs/`) — fine for the actual app, but pollutes it for a throwaway test run.
- `target/` stays inside the repo, gitignored; same ~10+ GB expectation as Windows.

## Known pitfalls
- `widget_rects` based tests find buttons by size; a wide (named) row doesn't match a 30×30 / 36×29 filter, so those tests keep the default width.
- A `ui.horizontal` row registers a second rect a few points wider than its button: de-duplicate by `top()` in tests.
- Passing `ui` and `ui.id()...` to a free function in one call trips the borrow checker: compute the id first.

- Checking UI in the running app: start it with `VECTORCRAFT_NO_PREFS=1` and `--control 7979` (no preferences are read or written), then `ui.drag` / `ui.screenshot` over the control channel. Synthetic input goes one event per frame; queue one drag at a time.

## To do
- The bottom controls of a wide Tools panel (fill/stroke, draw mode) are still centred at their old size.
- Consider a menu entry / command for the widths (`window.iconNames`) so the control channel and MCP can set them.
