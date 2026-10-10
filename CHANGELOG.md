（此前的历史未记录，当前状态见 CLAUDE.md）

## 2026-10-10
- 从 storytold/vectorcraft fork 并 clone 到 D:\Work\GitHub\vectorcraft；main 仅同步上游，自定义在 custom 分支。
- .gitignore 补充 .claude/、node_modules/。
- 功能：左侧工具栏、右侧图标列支持拖动边缘变宽，宽度 ≥ 110 时在图标旁显示中文名称（类 Illustrator），拖回变窄恢复纯图标。
  - 右侧图标列：拖左边缘（`dock.rs`，上限 240）；左侧工具栏：拖右边缘（`toolbar.rs`，上限 260）；仅停靠状态生效，浮动工具栏不变。
  - 新增 `UiState.icon_column_width` / `toolbar_width`（随偏好保存，默认 0 = 纯图标；不进 Workspace 快照，减少与上游的冲突面）。
  - 新增 `widgets::paint_icon_row`、`widgets::width_grip`、`widgets::LABELS_MIN`。
  - 测试：`dock::tests::dragging_the_column_edge_…`、`toolbar::tests::dragging_the_right_edge_…`、`an_unusable_saved_column_width_is_reset`；ui-egui 全部 885 测试通过，fmt / clippy -D warnings 干净。
- 构建环境：Rust 1.99 装在 `D:\Dev\rust`（RUSTUP_HOME / CARGO_HOME 为用户级变量），VS Build Tools 装在 `D:\Dev\BuildTools`（Windows SDK 在 C 盘）。编译命令与注意事项见 `CUSTOM.md`（单独文件，避免与上游的 `CLAUDE.md` 同步冲突）。
- 打包：Windows x64 便携 zip（`dist/release/vectorcraft-0.8.0-custom-windows-x64-portable.zip`，已被 gitignore）。
  - 命令同 `packaging/windows/package.ps1` 的构建部分：`cargo build --release --locked -p vectorcraft -p vectorcraft-cli --target x86_64-pc-windows-msvc`，环境变量 `CARGO_TARGET_X86_64_PC_WINDOWS_MSVC_RUSTFLAGS=-C target-feature=+crt-static`、`VECTORCRAFT_REQUIRE_WINRES=1`。
  - 未签名、未嵌入 craft-fonts（CJK 靠系统字体回退）；MSI 需要另装 .NET + WiX v5，暂未做。
- 发布：生成 MSI（WiX v5，`D:\Dev\dotnet`），连同便携 zip、SHA256SUMS 发布为 GitHub Release `v0.8.0-custom.1`（https://github.com/Krstic-c/vectorcraft/releases/tag/v0.8.0-custom.1）。
- 仓库：fork 默认分支改为 `custom`；删除 fork 上继承自上游的 235 个分支，只保留 `main`、`custom`（上游分支仍在 upstream 上）。
- 文档：`CUSTOM.md` 补充"与上游同步"（不要用网页 Sync fork）和"打包发布"流程。
