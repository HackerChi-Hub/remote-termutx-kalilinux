```markdown
# kalinethunter-remote (LF 版本)

> 适用于 Termux 中的 Kali Linux / Debian / Ubuntu 容器  
> Bash-only • 自动检测 CRLF • 颜色提示  
> 一键安装 XFCE 远程桌面（VNC + XRDP），自动切换清华源，配置中文环境 + fcitx5 输入法，并自带备份/恢复机制。

---

## 1. 使用场景说明：Termux + Kali Linux

本脚本在设计时就考虑了 **手机 / 平板上通过 Termux 跑 Kali / Debian 容器** 的场景，例如：

- Termux + `proot-distro` 安装的 Kali / Debian / Ubuntu 容器
- Termux + `Andronix` / `Nhterm` / 其它类似工具安装的 Linux 根文件系统

在这些场景下，本脚本可以在 **容器内部** 帮你配好：

- XFCE 桌面环境
- TigerVNC 服务
- XRDP（远程桌面协议）
- 中文字体 + 中文 locale
- fcitx5 中文输入法
- 一键启动/停止脚本（`vnc-start` / `vnc-stop` / `rdp-start` / `rdp-stop`）

> 注意：脚本是在 **Kali 容器里运行**，不是在 Android/Termux 主环境直接运行。

---

## 2. 前置准备（在 Termux 环境中）

### 2.1 安装 Termux 必要组件

在 Termux 中执行：

```bash
pkg update && pkg upgrade -y
pkg install -y proot-distro openssh
```

例如使用 `proot-distro` 安装 Kali：

```bash
proot-distro install kali
proot-distro login kali
```

进入 Kali 后，后续所有操作都在 **Kali 容器内部** 执行。

若你已经有 Kali 根文件系统（通过其它方式安装），只需要进入对应 chroot/proot 环境即可。

---

## 3. 在 Kali 容器中使用脚本

### 3.1 准备脚本文件

在 Kali 容器内：

1. 把脚本保存为 `kalinethunter-remote.sh`：

   ```bash
   nano kalinethunter-remote.sh
   # 或 vim / cat > kalinethunter-remote.sh 贴进去
   ```

2. 确保脚本行尾为 **LF**（Unix 换行）：

   ```bash
   apt update && apt install -y dos2unix
   dos2unix kalinethunter-remote.sh
   ```

3. 赋予执行权限：

   ```bash
   chmod +x kalinethunter-remote.sh
   ```

### 3.2 在 Kali 中启用 root / sudo

在多数 Termux 容器环境中，默认就是 root 用户（`id -u = 0`）：

- 若你运行 `id -u` 显示 `0`，说明已经是 root，可以直接用：
  ```bash
  ./kalinethunter-remote.sh
  ```

- 如果不是 root，请使用 `sudo` 或 `su -` 切换到 root 后再运行。

### 3.3 运行脚本

在 Kali 容器内执行：

```bash
./kalinethunter-remote.sh
```

启动后会出现菜单：

```text
==============================
  XFCE 远程环境配置脚本
==============================
请选择操作：
  1) 安装/配置（默认使用清华源）
  2) 恢复到最近一次备份
  3) 恢复到指定备份批次
  4) 退出
```

首次使用请选择 **`1) 安装/配置`**，按提示等待完成。

---

## 4. Termux 环境下的端口转发与连接方式

在 Android / Termux + Kali 容器场景下，**“宿主网络”是 Android，容器网络是 proot/chroot 内部”**，但我们常见的做法是：

- 从 **外部电脑** 连接到 **Termux/手机的某个端口**（通过局域网 IP 或 ADB 端口转发）
- 然后由 Termux 把这些端口转发 / 暴露到容器里相应服务（VNC/XRDP）

本脚本在容器内启动的服务监听的是 **容器内部的 `localhost`**（通常透传到 Termux 的 127.0.0.1 上），典型连接方式如下。

### 4.1 启动 VNC / XRDP 服务（在 Kali 容器内）

1. 切换到普通用户（建议不要长期用 root 跑桌面）：

   ```bash
   # 在容器内先创建用户（如第一次使用）
   adduser kali

   su - kali
   ```

2. 首次使用 VNC，设置密码：

   ```bash
   vncpasswd
   ```

3. 启动 VNC：

   ```bash
   vnc-start
   # 默认 :1 -> 5901，分辨率 1280x720
   ```

4. 启动 XRDP（在容器内 root 权限）：

   ```bash
   sudo rdp-start
   # 监听 0.0.0.0:3390（在容器内部）
   ```

### 4.2 从同一手机上的 VNC 客户端连接

若你在 **同一台 Android 设备** 上装了 VNC 客户端（如 `bVNC`, `VNC Viewer`）：

1. VNC 默认监听：`localhost:5901`（在容器 / Termux 内部）
2. 由于容器与 Termux 共享 127.0.0.1，一般可以直接在 VNC 客户端连接：

   - **地址**：`127.0.0.1`
   - **端口**：`5901`

如果你的容器或 Termux 使用了其它网络隔离方式，可能需要在 Termux 中转发端口，比如：

```bash
# 在 Termux 主环境中执行（不是在 Kali 容器里）
# 把 Termux 的 5901 转发到 Kali 容器（如果需要，这取决于你的 proot 启动方式）
# 常见 proot-distro 其实不需要额外映射，127.0.0.1 会贯通
```

> 实际情况通常是：**Termux 和容器共享 127.0.0.1**，所以你只要在容器中启动 VNC，就可以在 Android 上直接连 127.0.0.1:5901。

### 4.3 从 PC 通过 ADB 端口转发访问

若手机通过 USB 连接电脑，可用 ADB 转发端口：

1. 在 PC 上：

   ```bash
   adb forward tcp:5901 tcp:5901
   adb forward tcp:3390 tcp:3390
   ```

2. 保证在手机上（Termux + Kali 容器内）已经启动：

   ```bash
   # 容器里：
   vnc-start      # VNC :1 -> 5901
   sudo rdp-start # XRDP -> 3390
   ```

3. 在 PC 上：

   - VNC 客户端连接：`127.0.0.1:5901`
   - RDP 客户端连接：`127.0.0.1:3390`

ADB 会把数据从 PC 的本地端口转到手机上的对应端口。

### 4.4 从 PC 通过 Wi‑Fi / 局域网访问（不推荐裸奔）

1. 在 Android 端查看自己 IP（Termux）：

   ```bash
   ip addr show wlan0
   # 假设得到 192.168.1.123
   ```

2. 确保 VNC / XRDP 在容器内已启动。（如果容器网络与 Termux 同网段，端口会暴露在 Android 机器的 IP 上）

3. PC 端 VNC/RDP 客户端连接：

   - VNC：`192.168.1.123:5901`
   - RDP：`192.168.1.123:3390`

> 安全起见，不建议在公网裸奔暴露 5901 / 3390 端口；若必须远程访问，建议走 ADB / SSH 隧道或 VPN。

---

## 5. 在 Termux 场景下的性能与省电建议

移动设备性能有限，建议：

- 以较低分辨率启动 VNC：

  ```bash
  VNC_GEOMETRY=1024x600 vnc-start
  ```

- 在 XFCE 内关闭合成器：
  - 设置 → 窗口管理器微调 / 合成器 → 关闭窗体阴影、透明等效果。

- 在 Termux 主环境中：

  ```bash
  termux-wake-lock   # 防止休眠杀进程
  ```

- 使用 `tmux` 或 `screen` 管理容器中的会话，避免网络中断导致服务退出。

---

## 6. 备份与恢复（在 Kali 容器内）

备份目录和机制与普通 Debian/Ubuntu 完全一样：

- 备份根目录：

  ```text
  /var/backups/xfce-remote-setup/<RUN_ID>/
  ```

- 最近一次备份软链接：

  ```text
  /var/backups/xfce-remote-setup/latest
  ```

- 恢复方式：

  ```bash
  ./kalinethunter-remote.sh
  # 选择：2) 恢复到最近一次备份
  # 或    3) 恢复到指定备份批次（输入 RUN_ID）
  ```

恢复只影响 **容器内部的配置文件**，不会改动 Android / Termux 主系统。

---

## 7. 常见问题（特定于 Termux + Kali）

1. **脚本提示不是 root 用户**  
   - 检查 `id -u` 是否为 0。
   - 若不是，使用：

     ```bash
     su -
     # 或
     sudo bash
     ```

2. **启动 VNC 没有图形/黑屏**  
   - 确认在 **普通用户** 下运行 `vnc-start`（不要用 root 桌面）。
   - 删除旧 `.vnc` 配置再试：

     ```bash
     rm -rf ~/.vnc
     vncpasswd
     vnc-start
     ```

3. **XRDP 登录后黑屏**  
   - 检查 `/etc/xrdp/startwm.sh` 是否被脚本替换为使用 `startxfce4`。
   - 检查用户 `~/.xsession` 内容是否为：

     ```sh
     #!/bin/sh
     [ -f "$HOME/.xprofile" ] && . "$HOME/.xprofile"
     exec startxfce4
     ```

4. **Termux / 容器被系统杀掉**  
   - Android 为省电会杀后台进程，建议：
     - 给 Termux 允许“后台无限制”。
     - 使用 `termux-wake-lock`。
     - 尽量在充电状态下长时间使用图形桌面。

---

## 8. 简要功能回顾

脚本在 Kali / Debian / Ubuntu (含 Termux 容器) 中会自动完成：

- 清华源配置（若识别为 debian/ubuntu）
- 全系统升级
- XFCE + TigerVNC + XRDP 安装与配置
- 中文字体 + zh_CN.UTF-8 locale + fcitx5
- VNC / XRDP 环境变量（含输入法、音频）
- 一键脚本：`vnc-start`, `vnc-stop`, `rdp-start`, `rdp-stop`
- 所有关键配置文件的备份与可逆恢复

结合 Termux / proot-distro / ADB / Wi‑Fi，即可在手机上获得相对完善的 Kali 图形远程桌面环境。

---
```

如果你愿意，我也可以帮你写一份更“极简版”的 README，只保留和 Termux+Kali 直接相关的几条操作命令，方便你在手机上查看时快速上手。
