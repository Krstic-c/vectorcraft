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
