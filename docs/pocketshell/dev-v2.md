# PocketShell 二次开发、优化与定制版本部署

记录日期：2026-10-17。

- Fork：https://github.com/longcrawfish/Big-Pony-pocketshell
- 上游：https://github.com/Big-Pony/pocketshell
- 当前机器部署说明：[readme.md](./readme.md)
- 手机与墨水屏问题背景：[optimize-in-phone.md](./optimize-in-phone.md)

推荐路线：先完成墨水屏适配和 tmux 尺寸恢复，源码构建后手动部署；稳定后再建立自己的 Release 和在线升级流程。

本文是实施方案，不代表功能已开发或部署。此前核查只读取了文档、源码、本机服务配置和二进制版本，没有修改运行中的应用，也没有执行构建或测试。

## 1. 已核实的基线

| 项目 | 核查结果 |
| --- | --- |
| Fork 默认分支 | `main` |
| Fork 核查提交 | `8aa6474e81400fa1896369f3920dbffaa6465585` |
| Fork 项目版本 | `1.21.5` |
| Fork Release | 核查时尚无 Release |
| 本机二进制版本 | `1.21.5-292c71d` |
| systemd 配置文件 | `/etc/systemd/system/pocketshell.service`，已存在 |
| 服务启动路径 | `/opt/pocketshell/pocketshell-agent` |
| 命令链接 | `/usr/local/bin/pocketshell-agent` → `/opt/pocketshell/pocketshell-agent` |
| 服务用户 | `aiops` |
| 服务密钥与数据目录 | `/home/aiops/.pocketshell` |
| 监听地址 | 服务配置为 `127.0.0.1:8722` |
| 对外地址 | 服务配置为 `wss://pshell.taloseng.com` |

本机二进制与 fork HEAD 的构建 SHA 不同，不能仅因版本号一致就认为代码完全相同。正式修改前应记录当前版本，并尽可能确认其对应源码。

`readme.md` 中“服务尚未配置”的记录已落后于本次文件检查结果。但服务配置存在不等于服务正在运行；本次没有验证 systemd 运行状态、Caddy、HTTPS 或手机连接。

## 2. 项目结构与修改位置

| 目录 / 文件 | 用途 | 本次修改方向 |
| --- | --- | --- |
| `app/` | Svelte 5 + Vite + xterm.js 前端 | 字号、按钮、主题、墨水屏模式 |
| `app/src/components/SettingsPanel.svelte` | 设置界面 | 独立界面字号、终端字号范围 |
| `app/src/components/Keyboard.svelte` | 虚拟键盘 | 触摸尺寸与窄屏布局 |
| `app/src/components/StatusBar.svelte` | 状态栏 | 字号、对比度与操作布局 |
| `app/src/components/TopTabs.svelte` | 标签栏 | 可读性与触摸操作 |
| `app/src/lib/settings.ts` | 设备本地偏好 | 保存墨水屏模式、界面字号 |
| `app/themes/` | 内置主题 | 高对比浅色主题 |
| `agent/src/terminal.ts` | tmux / 终端管理 | 手机释放尺寸控制 |
| `agent/src/config.ts`、`agent/src/update-*.ts` | 配置与更新 | Fork 更新来源和信任策略 |
| `agent/scripts/build-bin.ts` | 打包 | 将前端嵌入 Agent |
| `scripts/release.sh` | 发布 | 压缩包、校验和、标签、GitHub Release |

生产环境交付的是包含前端资源的 Agent 单文件二进制。修改 CSS 或 Svelte 组件后，需要重新打包 Agent；仅把 `app/dist` 或 CSS 文件放进当前安装目录不能完成升级。

## 3. 优化范围与实施顺序

### 3.1 第一阶段：墨水屏阅读与触摸体验

目标设备为 Bigme HiBreak S Color，浏览器为 Brave Nightly。以下尺寸是待真机验证的起点，不是已验收参数。

1. 增加独立“界面字号”，与终端字号分开。
2. 设置标签、菜单和主要按钮文字以 16px 为起点，辅助文字以 14px 为起点。
3. 主要触摸目标至少 44px；虚拟键盘结合屏幕宽度单独设计，不机械地给每个键套用相同宽度。
4. 增加墨水屏模式：白底、深色文字、清晰边框，减少阴影、透明效果、闪烁和装饰动画。
5. 终端字号上限可试验性提高到 24，同时验证窄屏换行、键盘展开及横竖屏切换。
6. 墨水屏模式按设备保存，避免电脑端被强制使用同一布局。

不要只修改 `body` 字号，组件内的固定字号会覆盖继承值。不要使用全局 `font-size: ... !important` 或整体 `transform: scale(...)` 代替布局适配。

新增内置主题时，除主题文件外，还要核对 `settings.ts` 中的主题类型与白名单、生成流程及界面选项。终端内工具输出的 RGB 真彩色背景不一定受主题调色板控制，需要单独验证。

### 3.2 第二阶段：tmux 尺寸控制生命周期

当前 `agent/src/terminal.ts` 调用了 `tmux resize-window`。该操作会使窗口进入手动尺寸模式，手机离开后可能继续保持窄屏。现有手动恢复方法见 [手机优化文档第 7 节](./optimize-in-phone.md#7-手机使用结束后释放-tmux-尺寸控制)。

建议实现：

1. 手机接管窗口时，保存原窗口尺寸策略，包括是否继承全局设置。
2. 增加明确的“释放尺寸控制”操作。
3. 最后一个持有控制权的连接释放后，恢复原策略。
4. 区分主动释放、临时断线与重连，避免网络抖动反复改变尺寸。
5. 多客户端连接时确定唯一尺寸控制者，防止手机与电脑争抢尺寸。
6. 对窗口关闭、连接异常及 Agent 重启明确恢复策略。

这部分需要后端状态和回归测试，不能仅依赖网页关闭事件。恢复时也不能一律覆盖为 `latest`，应尊重原有窗口配置。

### 3.3 第三阶段：性能优化

- 先测量长输出、滚动、重连和后台恢复，再决定是否调整渲染或批处理。
- 墨水屏模式优先减少无意义重绘和动画，同时检查输入延迟。
- 保留上游已有的 xterm.js 补丁、重放和流控机制。
- 框架、Bun、xterm.js 的升级与业务改动分开进行，便于定位回归。

## 4. 开发目录与分支

`talk-dev` 保留运维文档，应用源码放在独立 Git 仓库。在选择好的开发父目录执行：

```bash
git clone https://github.com/longcrawfish/Big-Pony-pocketshell.git
cd Big-Pony-pocketshell

git remote add upstream https://github.com/Big-Pony/pocketshell.git
git switch -c feat/eink-ui
```

若已有源码目录或 `upstream`，先检查现状，不重复添加。建议将界面适配、tmux 修复、发布流程拆成独立提交或分支。

后续获取上游：

```bash
git fetch upstream --tags
```

在独立集成分支合并上游、解决冲突并验证后，再进入定制主分支；不要直接用上游覆盖定制修改。

## 5. 开发运行

项目说明要求 Bun ≥ 1.3。建议固定经过验证的具体 Bun 版本，在本机和 CI 中保持一致。`app/` 与 `agent/` 是两个独立项目，根目录没有 `package.json`。

从 fork 根目录安装依赖：

```bash
cd app
bun install --frozen-lockfile

cd ../agent
bun install --frozen-lockfile
```

使用独立端口和密钥目录运行开发 Agent：

```bash
# 在 agent/ 目录执行
POCKETSHELL_HOST=127.0.0.1 \
POCKETSHELL_PORT=18722 \
POCKETSHELL_ADVERTISE=ws://127.0.0.1:18722 \
POCKETSHELL_KEY_DIR="$HOME/.pocketshell-dev" \
POCKETSHELL_UPDATE=0 \
bun run start
```

另开终端，在 fork 的 `app/` 目录执行：

```bash
bun run dev
```

电脑访问 `http://localhost:5173`，使用开发 Agent 的配对信息连接。真机测试建议使用独立 HTTPS 测试域名反代开发实例，并相应设置开发实例的 `POCKETSHELL_ADVERTISE` 为该测试域名的 `wss://` 地址；正式域名继续提供稳定版本。

独立密钥目录隔离配置与配对身份，但同一系统用户下的 tmux 会话仍可能共享。测试尺寸控制时创建专用测试会话，避免接管正在工作的会话。

## 6. 测试与构建

分别在两个项目目录执行检查：

```bash
# app/ 目录
bun run typecheck
bun run test
```

```bash
# agent/ 目录
bun run typecheck
bun test
```

完整测试还涉及 `tmux`、`libarchive-tools` 和字体测试所需的 `uv`，以 `.github/workflows/ci.yml` 为准。当前 CI 没有执行需要真实 Agent 与 tmux 的全部浏览器端到端测试，不能把 CI 通过等同于真机验收通过。

修改验收后先提交，再构建：

```bash
# agent/ 目录
bun run build:bin

./dist/pocketshell-agent-linux-x64 --version
```

`build:bin` 会自动构建前端、生成嵌入资源，并编译 Linux/macOS 多种架构。本机使用：

```text
agent/dist/pocketshell-agent-linux-x64
```

先提交再构建很重要：前端缓存版本包含 Git SHA。同一提交上反复构建未提交修改，不能可靠地产生新的缓存版本。`app/package.json` 与 `agent/package.json` 的版本号应保持一致。

## 7. 部署到当前机器

### 7.1 保留现有入口与身份

目标链路：

```text
手机 → https://pshell.taloseng.com → Caddy → 127.0.0.1:8722
                                                ↓
                              /opt/pocketshell/pocketshell-agent
```

保留域名、服务用户和 `/home/aiops/.pocketshell`，在数据格式兼容的前提下通常无需重新配对。HTTPS 继续由 Caddy 处理，无需为界面改动重新配置域名或证书。

正式实施前确认服务路径与状态：

```bash
sudo systemctl show pocketshell -p User -p ExecStart -p ActiveState
/opt/pocketshell/pocketshell-agent --version
```

以下命令针对已核查的 `aiops` 用户、目录和服务名。若现场配置不同，先调整路径，不直接套用。

### 7.2 暂时关闭上游在线更新

执行：

```bash
sudo systemctl edit pocketshell
```

在 drop-in 中添加以下配置；若已有其他覆盖项，保留它们：

```ini
[Service]
Environment=POCKETSHELL_UPDATE=0
```

部署时重载配置并重启服务后生效。这能避免手动构建的定制版被应用内上游更新覆盖。

### 7.3 备份与替换

在 fork 仓库根目录执行，先确认新二进制能够启动并输出预期版本：

```bash
./agent/dist/pocketshell-agent-linux-x64 --version
```

安排短暂维护窗口。服务重启会中断连接，应避开关键操作，升级后验证任务恢复。以下步骤逐段执行，每一步成功后再继续；若备份失败，不替换二进制。

```bash
sudo systemctl stop pocketshell

backup_dir="/home/aiops/pocketshell-backups/$(date +%Y%m%d-%H%M%S)"
install -d -m 700 "$backup_dir"

sudo cp -a /opt/pocketshell/pocketshell-agent "$backup_dir/"
sudo cp -a /etc/systemd/system/pocketshell.service "$backup_dir/"
sudo cp -a /home/aiops/.pocketshell "$backup_dir/"

# 保存实际使用的 drop-in，包括本次禁用 OTA 的设置
if [ -d /etc/systemd/system/pocketshell.service.d ]; then
  sudo cp -a /etc/systemd/system/pocketshell.service.d "$backup_dir/"
fi

printf '备份目录：%s\n' "$backup_dir"
```

记录输出的备份目录。备份包含配对密钥，保持私有，不要提交 Git。

```bash
sudo install -o aiops -g aiops -m 755 \
  agent/dist/pocketshell-agent-linux-x64 \
  /opt/pocketshell/pocketshell-agent.new

sudo mv /opt/pocketshell/pocketshell-agent.new \
  /opt/pocketshell/pocketshell-agent

sudo systemctl daemon-reload
sudo systemctl start pocketshell
```

通过同目录临时文件替换实际启动文件，保留 `/usr/local/bin/pocketshell-agent` 的现有符号链接。不要只更新 `~/.local/bin/pocketshell-agent`，因为当前 systemd 服务不从该路径启动。

### 7.4 验收

```bash
/opt/pocketshell/pocketshell-agent --version
sudo systemctl status pocketshell --no-pager
curl -I http://127.0.0.1:8722/
curl -I https://pshell.taloseng.com/
```

需要排查时在本机查看服务日志，避免公开粘贴可能含配对信息的完整日志。

| 场景 | 通过标准 |
| --- | --- |
| 页面与版本 | 普通标签页、桌面 PWA 均取得新资源，版本符合预期 |
| 身份与配置 | 原设备可连接，本地偏好和服务数据仍可用 |
| 阅读体验 | 中文、英文、背景色块和辅助文字清晰 |
| 布局 | 按钮可点击，菜单可滚动，键盘不遮挡输入区 |
| 输入 | 中文输入法、快捷键、删除和粘贴正常 |
| 重连 | 切后台、断网恢复后，终端可继续使用，输出恢复正常 |
| tmux 尺寸 | 释放控制后电脑可恢复尺寸，多客户端不反复争抢 |
| 通知 | 若本次涉及相关逻辑，验证后台通知和返回对应会话 |

更新 PWA 时先正常刷新、关闭重开，不要默认清空站点数据，否则可能丢失本地身份和偏好。

### 7.5 回滚

若新版本异常，在终端将 `backup_dir` 设置为第 7.3 节实际记录的备份目录，然后执行：

```bash
sudo systemctl stop pocketshell

sudo install -o aiops -g aiops -m 755 \
  "$backup_dir/pocketshell-agent" \
  /opt/pocketshell/pocketshell-agent.rollback

sudo mv /opt/pocketshell/pocketshell-agent.rollback \
  /opt/pocketshell/pocketshell-agent

sudo systemctl start pocketshell
/opt/pocketshell/pocketshell-agent --version
sudo systemctl status pocketshell --no-pager
```

纯界面修改通常不需要恢复数据目录。若升级包含不兼容的数据迁移，应在升级前设计并验证数据回滚方案，不能直接用旧备份覆盖升级后的新数据。保持 OTA 关闭，直到确认更新来源和版本策略。

## 8. 发布自己的 Release

Fork 只有源码还不够，安装脚本需要下载对应 Release 的二进制资产。建议保持当前资产命名：

```text
pocketshell-agent-linux-x64.tar.gz
pocketshell-agent-linux-arm64.tar.gz
pocketshell-agent-darwin-arm64.tar.gz
pocketshell-agent-darwin-x64.tar.gz
SHA256SUMS.txt
```

每个压缩包保留 `LICENSE` 和 `THIRD-PARTY-FONTS.md`。上游 `scripts/release.sh` 已包含打包逻辑，但用于 fork 前建议调整：

1. `gh release create` 和 `gh release view` 显式指定 `--repo longcrawfish/Big-Pony-pocketshell`，避免仓库推断歧义。
2. 将版本提交放在构建之前，使构建 SHA 与发布标签对应。
3. 删除不适用的固定作者信息。
4. 固定 Bun 版本，并让发布以前置测试通过为条件。
5. 同步维护两个 `package.json` 的版本号，采用递增的三段数字版本。

当前发布脚本只接受如 `1.21.6` 的三段数字版本；现有 OTA 比较逻辑也主要比较三个数字段。若要引入 `-fork.1` 等版本方案，需要一并修改并测试版本解析与发布逻辑，不能直接套用。

当前脚本正式运行会修改版本、构建、提交、打标签、推送并创建 GitHub Release。`--dry-run` 只构建打包，不提交或发布，且不会把传入的新版本号写进产物。不要把 dry-run 产物误当作已完成版本升级的正式包。

推荐最终发布顺序：修改版本 → 提交 → 测试与构建 → 核验产物版本 → 打标签 → 发布资产与校验和。先在自己的仓库中完善脚本，再执行正式发布。

## 9. 新机器安装定制版

以下示例假设将来已经发布 `v1.21.6`，不是当前可直接下载的版本。执行时替换为实际发布版本。

```bash
curl -fsSL \
  https://raw.githubusercontent.com/longcrawfish/Big-Pony-pocketshell/main/install.sh \
  -o /tmp/pocketshell-fork-install.sh

less /tmp/pocketshell-fork-install.sh

REPO=longcrawfish/Big-Pony-pocketshell \
VERSION=1.21.6 \
sh /tmp/pocketshell-fork-install.sh
```

必须明确指定 `REPO`：脚本来自 fork，并不意味着它默认下载 fork 的二进制；核查时脚本默认仍指向 `Big-Pony/pocketshell`。版本与仓库变量要传给执行脚本的 `sh`，而不是只传给下载脚本的 `curl`。

以普通用户运行安装脚本，不加 sudo。该步骤只安装用户目录中的二进制，注册服务还需执行 `install` 子命令。例如在与当前机器相同的用户和域名配置下：

```bash
sudo /home/aiops/.local/bin/pocketshell-agent install \
  --advertise wss://pshell.taloseng.com \
  --name talos-dev \
  --user aiops \
  --host 127.0.0.1 \
  --port 8722
```

全新机器的域名、用户、Caddy、HTTPS 和配对按 [部署文档](./readme.md) 配置。其他机器应使用各自的用户和访问地址。现有服务升级优先使用第 7 节的备份替换流程，无需每次重新注册服务。

## 10. 后续接入自己的在线升级

当前源码有两层控制：

- `POCKETSHELL_UPDATE_REPO` 决定检查哪个仓库的 Release。
- `agent/src/config.ts` 中的 `OFFICIAL_UPDATE_REPO` 与 `official` 判定限制允许执行更新的来源。

因此，仅设置：

```ini
Environment=POCKETSHELL_UPDATE_REPO=longcrawfish/Big-Pony-pocketshell
```

在当前代码中只能检查 fork 的新版本，不能应用更新。应以源码为准，不把部署文档中“可指向 fork”的描述理解为已经支持 fork 自动安装。

要支持自己的 OTA，需要在定制版本中将内置可信来源明确设为自己的仓库，并测试对应的配置、检查和应用流程。保留来源校验，不要简单删除限制。新安装脚本的默认仓库、文档与相关测试也应同步更新。

启用前确认：

1. 已有包含正确平台资产与 `SHA256SUMS.txt` 的正式 Release。
2. 新版本号高于当前版本，前后端版本一致。
3. 服务用户可写二进制所在目录，支持临时文件与原子替换。
4. systemd 的 `Restart=always` 生效，更新后能可靠重启。
5. 升级和失败回滚已经在测试实例验证。

完成后再移除禁用 OTA 的配置，或明确设置 `POCKETSHELL_UPDATE=1`，重载 systemd 并重启。第一版建议继续手动部署，将在线升级改造作为独立阶段。

## 11. 源码参考

以下链接固定到本次核查提交，避免后续 `main` 变更影响复查：

- [项目说明与开发命令](https://github.com/longcrawfish/Big-Pony-pocketshell/blob/8aa6474e81400fa1896369f3920dbffaa6465585/README-CN.md)
- [构建脚本](https://github.com/longcrawfish/Big-Pony-pocketshell/blob/8aa6474e81400fa1896369f3920dbffaa6465585/agent/scripts/build-bin.ts)
- [前端构建与缓存版本](https://github.com/longcrawfish/Big-Pony-pocketshell/blob/8aa6474e81400fa1896369f3920dbffaa6465585/app/vite.config.ts)
- [tmux 管理](https://github.com/longcrawfish/Big-Pony-pocketshell/blob/8aa6474e81400fa1896369f3920dbffaa6465585/agent/src/terminal.ts)
- [配置与更新来源限制](https://github.com/longcrawfish/Big-Pony-pocketshell/blob/8aa6474e81400fa1896369f3920dbffaa6465585/agent/src/config.ts)
- [安装脚本](https://github.com/longcrawfish/Big-Pony-pocketshell/blob/8aa6474e81400fa1896369f3920dbffaa6465585/install.sh)
- [发布脚本](https://github.com/longcrawfish/Big-Pony-pocketshell/blob/8aa6474e81400fa1896369f3920dbffaa6465585/scripts/release.sh)
- [CI 检查](https://github.com/longcrawfish/Big-Pony-pocketshell/blob/8aa6474e81400fa1896369f3920dbffaa6465585/.github/workflows/ci.yml)
