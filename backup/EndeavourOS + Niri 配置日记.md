# 安装EndeavourOS 

- 安装前要记得更改镜像源
- 安装后记得 `sudo pacman -S timeshift` 备份系统, 无脑下一步就好, 然后cteate一下

# 安装niri
## niri本体
```sh
sudo pacman -S niri
```
## 安装其他组件
```sh
sudo pacman -S waybar foot xdg-desktop-portal-wlr wl-clipboard mako \
 network-manager-applet wofi swaybg brightnessctl playerctl
```
### 软件包功能说明
| 包名 | 主要功能 | 说明 |
|------|-----------|------|
| **waybar** | 状态栏 | 显示时间、电量、音量、网络状态等，类似于 Polybar 的 Wayland 版。 |
| **foot** | 终端模拟器 | 轻量、快速、原生 Wayland 终端，适合 Niri / Sway 等桌面。 |
| **xdg-desktop-portal-wlr** | 桌面集成接口 | 提供截图、屏幕共享、文件选择等 Wayland 桌面通用功能（wlr 专用实现）。 |
| **wl-clipboard** | 剪贴板工具 | 提供 `wl-copy` 与 `wl-paste` 命令，Wayland 下的剪贴板支持。 |
| **mako** | 通知服务 | Wayland 下的轻量通知守护进程，用于弹出系统消息。 |
| **network-manager-applet** | 网络托盘图标 | 图形化网络管理小程序，显示在 Waybar 或系统托盘上。 |
| **wofi** | 应用启动器 | 类似 rofi 的 Wayland 版本，用于搜索并启动应用程序。 |
| **swaybg** | 设置桌面壁纸 | 为 Wayland 桌面设置背景图片的小工具。 |
| **brightnessctl** | 屏幕亮度控制 | 调整笔记本或显示器亮度的命令行工具。 |
| **playerctl** | 媒体控制 | 控制音乐播放器（播放、暂停、下一首等），Waybar 常用插件依赖。 |

### 提示