# PocketShell：当前开发机部署说明

项目：https://github.com/Big-Pony/pocketshell

通过手机浏览器操作当前 Linux 开发机上的终端。采用 Caddy 提供 HTTPS，PocketShell 只监听本机，由 systemd 管理常驻服务。

## 当前状态与目标配置

记录日期：2026-10-17。以下步骤中，只有 PocketShell 二进制安装已经完成；服务、Caddy、HTTPS 和手机连接尚未配置或验收。

| 项目 | 配置或检查结果 |
| --- | --- |
| 系统 | Debian GNU/Linux 13（trixie），x86_64 |
| 开发用户 / 服务用户 | `aiops` |
| 已安装依赖 | `tmux`、`git`、`curl` |
| 已安装 PocketShell | `1.21.5-292c71d`，下载时已通过 SHA256 校验 |
| 初始二进制路径 | `/home/aiops/.local/bin/pocketshell-agent` |
| 手机访问地址 | `https://pshell.taloseng.com` |
| 对外 WebSocket 地址 | `wss://pshell.taloseng.com` |
| Agent 监听地址 | `127.0.0.1:8722` |
| DNS | 用户已确认域名解析到本机公网 IP；部署时仍需检查解析与公网可达性 |
| 端口与反代 | 检查时 80、443、8722 空闲，未安装 Caddy/Nginx |

访问链路：

```text
手机浏览器 / PWA
  → https://pshell.taloseng.com:443
  → Caddy（HTTPS、WebSocket 反向代理）
  → http://127.0.0.1:8722
  → PocketShell（aiops 用户）→ tmux → 开发工具
```

下面的命令在这台开发机上以 `aiops` 登录后执行。涉及系统配置的命令需要在终端输入 sudo 密码。主机需要持续开机、联网并避免休眠；手机断线不等于主机重启，后者不会保留正在运行的任务。

## 1. 检查 DNS 与网络

```bash
getent ahosts pshell.taloseng.com
sudo ss -ltnp
```

确认 DNS 指向本机的公网地址。如果配置了 AAAA 记录，也要确保对应 IPv6 地址可达。主机防火墙及云平台安全组需允许入站 TCP 80、443；如果公网 IP 在路由器上，需要将这两个端口转发到开发机。

8722 仅供本机反向代理访问，无需开放公网入口。保留现有 SSH 规则，避免修改防火墙时中断远程管理。Caddy 通过域名自动申请、续期证书，并将 HTTP 重定向到 HTTPS，相关前提见 [Caddy Automatic HTTPS](https://caddyserver.com/docs/automatic-https)。

## 2. 安装 PocketShell（本机已完成，可跳过）

全新机器可使用以下命令；普通用户执行安装脚本，不加 sudo：

```bash
sudo apt update
sudo apt install -y tmux git curl ca-certificates

curl -fsSL https://raw.githubusercontent.com/Big-Pony/pocketshell/main/install.sh \
  -o /tmp/pocketshell-install.sh
less /tmp/pocketshell-install.sh
sh /tmp/pocketshell-install.sh

/home/aiops/.local/bin/pocketshell-agent --version
```

脚本下载执行时的最新发行版，所以将来重新安装的版本可能与上面的记录不同。AI 命令行工具需另外以 `aiops` 用户安装、完成登录。

## 3. 安装并启动 PocketShell 系统服务

```bash
sudo /home/aiops/.local/bin/pocketshell-agent install \
  --advertise wss://pshell.taloseng.com \
  --name talos-dev \
  --user aiops \
  --host 127.0.0.1 \
  --port 8722
```

显式指定 `aiops`，让远程终端使用该用户的项目与开发环境。首次执行使用二进制完整路径，避免 sudo 的 PATH 找不到用户目录中的程序。安装器会注册并启动 systemd 服务，同时设置开机启动。

```bash
sudo systemctl status pocketshell --no-pager
sudo systemctl is-enabled pocketshell
sudo systemctl show pocketshell -p User -p ExecStart
sudo journalctl -u pocketshell -n 80 --no-pager
curl -I http://127.0.0.1:8722/
```

预期服务为 `active`、开机启动为 `enabled`、运行用户为 `aiops`，本机 HTTP 请求可以取得页面响应。先解决这里的启动错误，再配置外网入口。

保持 Agent 内置 TLS 关闭，HTTPS 由 Caddy 处理。不要设置 `POCKETSHELL_TLS=1`。

## 4. 安装 Caddy

使用 [Caddy 官方 Debian 稳定版软件源](https://caddyserver.com/docs/install#debian-ubuntu-raspbian)。以下为首次添加该软件源的步骤：

```bash
sudo apt update
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl gnupg ca-certificates

curl -fsSL https://dl.cloudsmith.io/public/caddy/stable/gpg.key \
  -o /tmp/pocketshell-caddy-stable.gpg.key
sudo gpg --dearmor \
  -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg \
  /tmp/pocketshell-caddy-stable.gpg.key

curl -fsSL https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt \
  -o /tmp/pocketshell-caddy-stable.list
sudo install -m 644 /tmp/pocketshell-caddy-stable.list \
  /etc/apt/sources.list.d/caddy-stable.list
sudo chmod a+r /usr/share/keyrings/caddy-stable-archive-keyring.gpg

sudo apt update
sudo apt install -y caddy
```

## 5. 配置域名与 HTTPS

先备份配置，然后编辑：

```bash
sudo cp -a /etc/caddy/Caddyfile "/etc/caddy/Caddyfile.backup.$(date +%Y%m%d-%H%M%S)"
sudoedit /etc/caddy/Caddyfile
```

对于本次全新安装，将默认示例站点替换为以下内容。如果将来已有其他站点，应保留它们，只增加下面的站点块：

```caddyfile
pshell.taloseng.com {
    reverse_proxy 127.0.0.1:8722
}
```

验证配置，只有验证成功才启用并重载：

```bash
sudo caddy validate --config /etc/caddy/Caddyfile --adapter caddyfile && \
sudo systemctl enable --now caddy && \
sudo systemctl reload caddy
```

检查服务、证书申请和 HTTPS：

```bash
sudo systemctl status caddy --no-pager
sudo journalctl -u caddy -n 100 --no-pager
curl -I https://pshell.taloseng.com/
```

首次申请证书可能需要稍等。HTTPS 检查不要添加 `-k`，以便发现证书问题。Caddy 原生支持 WebSocket 转发，无需额外添加 Nginx 风格的 Upgrade 配置。

## 6. 手机配对与通知

服务安装后使用 `/usr/local/bin/pocketshell-agent`。在开发机以 `aiops` 用户执行，不要加 sudo；设备管理必须与服务使用同一密钥目录：

```bash
POCKETSHELL_KEY_DIR=/home/aiops/.pocketshell \
  /usr/local/bin/pocketshell-agent pair --name my-phone
```

1. 手机打开 `https://pshell.taloseng.com`。
2. 粘贴终端输出的配对串，完成设备配对。配对码有效期为 5 分钟，过期后重新执行上述命令。
3. 将网页添加到主屏幕，之后从该图标打开。
4. 在应用设置里开启所用开发工具的通知，并允许浏览器/系统推送。iPhone 的 Web Push 需要使用添加到主屏幕的 PWA。

配对串、密钥目录和可能包含配对串的日志不要提交到仓库。已配对设备能够以 `aiops` 的权限操作主机。

## 7. 验收

页面能打开不代表终端和断线恢复已经通过验证，完成以下操作后才算部署成功：

1. 手机关闭 Wi-Fi，使用移动网络打开域名并连接终端，确认公网 HTTPS 和 WebSocket 均可用。
2. 在手机的持久终端会话中运行下面的命令，临时断网，再恢复连接，确认输出补齐且任务继续运行：

   ```bash
   for i in $(seq 1 30); do date; sleep 2; done
   ```

3. 进入一个实际项目，启动已安装的开发工具，确认其命令、登录状态和项目权限正常。
4. 开启通知后切到后台或锁屏，让一个短任务完成，确认能收到通知并返回对应会话。

## 8. 日常维护与排查

```bash
# 服务状态与日志
sudo systemctl status pocketshell caddy --no-pager
sudo journalctl -u pocketshell -n 100 --no-pager
sudo journalctl -u caddy -n 100 --no-pager

# 需要时重启 PocketShell
sudo systemctl restart pocketshell

# 查看已配对设备（aiops 用户）
POCKETSHELL_KEY_DIR=/home/aiops/.pocketshell \
  /usr/local/bin/pocketshell-agent devices list
```

撤销手机授权时，把下面的占位符替换为设备列表中的实际指纹：

```bash
POCKETSHELL_KEY_DIR=/home/aiops/.pocketshell \
  /usr/local/bin/pocketshell-agent devices remove <设备指纹>
```

| 现象 | 排查方向 |
| --- | --- |
| 域名打不开 | DNS、AAAA 记录、入站 80/443、安全组、路由器转发、Caddy 状态 |
| HTTPS 证书错误 | 查看 Caddy 日志，检查 DNS 和证书验证端口的公网可达性 |
| 返回 502 | 检查 PocketShell 服务，以及 `curl -I http://127.0.0.1:8722/` 是否成功 |
| 页面能开，终端一直重连 | 检查服务对外地址是否为 `wss://pshell.taloseng.com`，以及链路中的代理是否允许 WebSocket |
| 配对失败 | 重新生成配对串；确认命令和服务使用相同用户、相同密钥目录 |
| 找不到开发工具 | 在远程终端检查 PATH；确认工具安装在 aiops 用户环境并已登录 |
| 收不到通知 | 检查应用内工具通知开关、系统权限、PWA 安装情况和手机推送网络 |

## 参考

- [PocketShell 项目与安装说明](https://github.com/Big-Pony/pocketshell)
- [PocketShell 部署指南](https://github.com/Big-Pony/pocketshell/blob/main/DEPLOYMENT.md)
- [Caddy 安装说明](https://caddyserver.com/docs/install)
- [Caddy 自动 HTTPS](https://caddyserver.com/docs/automatic-https)
