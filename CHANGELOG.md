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
- Mac 上 clone 到 `~/Documents/GitHub/vectorcraft`（独立 git 工作区）：Windows 端的 2 个 commit 已经通过 git 带过来，不需要重新实现。装了 Homebrew Rust 1.99 验证：`cargo test -p vectorcraft-ui-egui` 885 全过、clippy/fmt 干净，实际跑起来用控制通道拖动验证了图标列展开/收起。`CUSTOM.md` 补了对应的 macOS 构建环境小节。
- 打包：macOS Apple Silicon（`packaging/macos/package.sh --arch aarch64`），ad-hoc 签名（没有 Apple 开发者证书，未 notarize）。发布为 GitHub Release `v0.8.0-custom.2`（https://github.com/Krstic-c/vectorcraft/releases/tag/v0.8.0-custom.2）：`.dmg` + CLI zip + SHA256SUMS。未 notarize 的后果：别的 Mac 下载后 Gatekeeper 会拦截，需要 `xattr -d com.apple.quarantine` 或系统设置里手动放行，release notes 里写了步骤。
- CI：fork 继承了上游完整的 `.github/workflows/release.yml`（macOS/Windows×3/Linux×2/Flatpak/FreeBSD/Web 全平台）。`macos` job 原本写死 `macos-15-xlarge`（付费大规格 runner，个人账号公共仓库免费额度不包含），改成 `macos-14`（标准免费 runner）后用 `workflow_dispatch` 在 `custom` 分支手动跑了一次：12 个 job 全部成功，自动建了草稿 release，改名发布为 `v0.8.0-custom.3`（https://github.com/Krstic-c/vectorcraft/releases/tag/v0.8.0-custom.3），23 个文件，全平台 + 内置 craft-fonts（CI 设了 `CRAFT_FONTS_DIR`，比本机手打的包中日文显示更完整）。同样未签名（没配 APPLE_CERTIFICATE / WINDOWS_CERTIFICATE / AZURE_* 这些 secrets）。
- 清理：本地 `target/` 编译缓存 15GB 删掉（`rm -rf target`），释放到 33GB 可用空间。确认过云端 CI 用的是 GitHub 自己的 `Swatinem/rust-cache`，跟本地 `target/` 完全独立，删了不影响云端编译；只是下次本机 `cargo build/test/run` 要全量重编译。`dist/release/`（46MB，已发布产物的本地副本）暂时保留。
