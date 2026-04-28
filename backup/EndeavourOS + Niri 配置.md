# 安装EndeavourOS 

- 安装前要记得更改镜像源, 且确认yay已配置可用, EndevourOS一般默认支持.
- 安装后记得 `sudo pacman -S timeshift` 备份系统, 无脑下一步就好, 然后cteate一下

# 安装niri
## niri本体
```bash
yay -S niri
niri --version # 本文编写时, 安装的niri版本为 "niri 25.08 (01be0e6)" , 不同版本的niri配置文件可能略有不同
```
## 安装其他组件
```bash
sudo pacman -S alacritty fuzzel waybar swaybg swaylock otf-font-awesome
```
### 软件包功能说明
| 包名 | 主要功能 | 说明 |
|------|-----------|------|
| **alacritty** | 终端模拟器 | 高性能 GPU 加速的跨平台终端，启动快、渲染流畅，适合日常开发和 Wayland 桌面环境使用。 |
| **fuzzel** | 应用启动器 / 菜单 | Wayland 下的轻量级快速启动器，可通过键盘搜索并启动应用程序，类似 rofi / wofi。 |
| **waybar** | 状态栏 | 显示时间、电量、音量、网络状态等信息，可高度自定义，Wayland 下类似 Polybar。 |
| **swaybg** | 设置桌面壁纸 | 为 Sway / Wayland 桌面设置背景图片的小工具，支持多显示器。 |
| **swaylock** | 锁屏工具 | Wayland 下的屏幕锁定程序，支持图像背景、屏幕模糊和自定义锁屏界面。 |
| **otf-font-awesome** | 字体图标库 | 提供 Font Awesome OpenType 图标字体，可在 Waybar、应用或自定义界面中显示图标。 |

> 安装`alacritty`是为了使用`niri`默认配置的快捷键`super+T`来打开其预配置的终端(`alacritty`), 可以根据个人需求替换成其他终端。通过修改`~/.config/niri/config.kdl`文件中的`binds{...}`块内部进行替换。
>
> 例入, 本文尝试使用`foot`作为默认终端 - TODO_后续单独开一个页面笔记记录`foot`配置
>> - 编辑上述文件内容
```kdl
binds {
    // Mod+T hotkey-overlay-title="Open a Terminal: alacritty" { spawn "alacritty"; }
    Mod+T hotkey-overlay-title="Open a Terminal: foot" { spawn "foot"; }
}
```
>> - 然后使用快捷键`super+T`打开的就是`foot`终端了, 但可能字体展示有些异常, 且终端会提示警告。
>> - foot 会检查字体是否为等宽字体（monospace）Noto Sans CJK JP 并不是等宽字体（monospace）
```bash
warning: foot: Noto Sans CJK JP: font does not appear to be monospace; check your config, or disable this warning by setting [tweak].font-monospace-warn=no
```
>> - 根据个人偏好, 安装喜欢的等宽字体解决
```bash
yay -S ttf-sarasa-gothic ttf-jetbrains-mono noto-fonts-emoji
```
>> - 编辑 `~/.config/foot/foot.ini` 以更换字体 (如果没有这个文件, 可以使用官方的模版`/etc/xdg/foot/foot.ini`快速进行修改)
```ini
[main]
include=/usr/share/foot/themes/ # 可选, 可配置自定义主题色, 这里使用https://github.com/catppuccin/foot
font=JetBrains Mono, Sarasa Mono SC, Noto Sans Mono CJK SC:size=24
pad=8x8 # 内边距=水平x垂直
dpi-aware=yes
[csd]
perferred=none # 隐藏窗口装饰烂
```
> 这里个人偏好打开终端就执行`fastfetch`一下, 编辑`~/.zshrc` 或 `~/.bashrc`
```bashrc
# Exe fastfetch after open bash
if command -v fastfetch >/dev/null 2>&1 && [[ $- == *i* ]]; then
  fastfetch
fi
```