# Arch 上的 Niri 之旅

> [!NOTE]
> 记录我用 Shorin-ArchLinux-Guide 一键脚本装好的 Shorin-Niri 桌面用法。其中一部分快捷键是脚本默认配置，一部分是我后来自己加改的，下文会标出来。

## 1. 一键安装脚本

我的系统是用 [Shorin-ArchLinux-Guide](https://github.com/SHORiN-KiWATA/Shorin-ArchLinux-Guide) 作者 shroinkiwata 的一键脚本装的，它能把刚装好的 Arch Linux 一键做成能用的状态，并装好桌面环境。

### 1.1 前提条件

- 安装一个 **btrfs 文件系统** 的 Arch Linux 系统
- 不需要任何准备工作，刚刚安装好的 Arch 就可以运行脚本

### 1.2 使用方法

在任意终端或者 TTY 运行以下命令：

```bash
curl -L shorin.xyz/archsetup | bash
```

运行中会弹个菜单让你选**桌面环境**，我选了 **Shorin-Niri**。

### 1.3 我的安装过程

我当时就选了 **Shorin-Niri**，其余可选模块基本没动、直接默认走完，装完自带一套配置完整的 Niri 桌面。

### 1.4 配置更新

Shorin-Niri 带了个 `shorinniri` 命令管理配置，可以 init 初始化、update 更新、remove 移除，操作前会把现有配置备份到 `~/.cache`，改坏了也能找回来。具体用法看命令的帮助信息即可，或参考原文的[「关于配置更新」一节](https://github.com/SHORiN-KiWATA/Shorin-ArchLinux-Guide/blob/main/wiki/archlinux/%E4%B8%80%E9%94%AE%E9%85%8D%E7%BD%AE%E6%A1%8C%E9%9D%A2%E7%8E%AF%E5%A2%83.md#%E5%85%B3%E4%BA%8E%E9%85%8D%E7%BD%AE%E6%9B%B4%E6%96%B0)。

### 1.5 失败回档

如果安装失败或者后悔了，可以回到安装前的时间点：

```bash
# 切换为 root
su -

# 回到安装前的时间点
shorin-undochange

# 回到安装桌面前的时间点
shorin-de-undochange
```

---

## 2. 装机后遇到的问题：输入法用不了

刚装好时，中州韵输入法直接"不可用"，所有应用都打不出中文。这个是我让 AI 帮忙解决的，AI 还在桌面留了份《输入法修复报告》，核心内容记在这。

### 2.1 原因

- 装的是作者仓库里的"增强版 fcitx5 核心"（`fcitx5-shorin-patched-git`），版本较旧
- 它内部删掉了一个叫 `xkbStateMask` 的函数，而中州韵插件恰好依赖这个函数
- 插件加载时"找不到函数"直接失败，所以中州韵用不了

### 2.2 修复过程

排查过程挺绕的：fcitx5 本身在运行，环境变量缺 `GTK_IM_MODULE`（在 `~/.config/niri/config.kdl` 环境变量区加了一行 `GTK_IM_MODULE "fcitx"`），中州韵词库目录是空的，手动重编译了一遍——但这些都没解决，最后深挖才发现中州韵插件根本没被加载：`librime.so` 依赖的 `xkbStateMask` 函数在新老版本里对不上。好在该增强版作者后来把修复合并回了最新版，**把核心升级到最新版即可**，不用改代码。

### 2.3 关键操作（重点）

用源码重新编译并安装了新的 fcitx5 核心：

```bash
cd /tmp/fcitx5-build && makepkg -si
```

新版：`5.1.20.r4146a2a-1`（旧版是 `5.1.20.r5.gd90014bd-1`）

> [!WARNING]
> 如果哪天系统更新后中州韵又"不可用"，很可能是 shorin-arch 仓库又发布了旧版本核心覆盖了手动装的新版。重新执行上面那条重新编译命令即可。安装包文件在 `/tmp/fcitx5-build/`，先别删，以后重装/回滚要用。

### 2.4 输入法皮肤：Ori-fcitx5

装好输入法后，我还换了输入法的皮肤和候选词字号。

> 项目地址：[Reverier-Xu/Ori-fcitx5](https://github.com/Reverier-Xu/Ori-fcitx5)，这是给 fcitx5 用的圆角主题，提供 OriDark（暗色）和 OriLight（亮色）两套。

皮肤文件安装到了用户目录（不是系统目录）：

```
~/.local/share/fcitx5/themes/OriDark/
├── highlight.svg
├── panel.svg
└── theme.conf
```

选用方式是在 fcitx5 的界面配置文件里把 `Theme` 指定为皮肤名。关键改动就两处（`~/.config/fcitx5/conf/classicui.conf`）：

```ini
# 使用 OriDark 皮肤
Theme=OriDark

# 候选词字号从默认的 12 调大到了 13
Font="Sans Serif 13"

# 小节其他配置参考：
# MenuFont="Sans Serif 10"        ← 菜单字号
# WheelForPaging=True             ← 鼠标滚轮翻页
# Vertical Candidate List=False   ← 垂直候选列表
EnableFractionalScale=True
```

> 顺带一提，`~/.local/share/fcitx5/themes/` 下还有一个 `Matugen` 主题，那是 Shorin-Niri 脚本带过来的，会跟着 Matugen 动态配色自动换肤。
>
> 输入法切换快捷键也在 fcitx5 的 `~/.config/fcitx5/config` 里配过：`Super+空格` 切换输入法，`Shift+Super+空格` 反向切换。

### 2.5 中州韵：小鹤双拼

我打字习惯用的小鹤双拼，配置在 rime 的用户目录 `~/.local/share/fcitx5/rime/` 下。做法是把 `double_pinyin_flypy` 设为默认方案——在 `default.custom.yaml` 的 `schema_list` 里把它挪到第一位：

```yaml
patch:
  schema_list:
    - schema: double_pinyin_flypy
    - schema: rime_ice
    - schema: luna_pinyin_simp
    - schema: wubi86
    - schema: bopomofo
  "menu/page_size": 6
```

另外 `melt_eng`（中英混输）和 `radical_pinyin`（笔画）这两个方案的配置文件里也加上了小鹤双拼的拼写派生规则（`speller/algebra` 指向 `algebra_double_pinyin_flypy`），切到这些方案时同样能按小鹤的键位习惯输入。

> 改完 rime 的配置文件，记得在 fcitx5 托盘右键「重新部署」才会生效。

### 2.6 当前状态

- fcitx5 核心升级后，中州韵插件已能正常加载
- 默认输入法就是中州韵，拼音方案用的是**小鹤双拼**
- 想切换输入法/开关中英文：`Ctrl+空格`

---

## 3. Shorin-Niri 特色快捷键（脚本自带默认配置）

### 3.1 实用工具类

| 快捷键 | 功能 | 说明 |
|--------|------|------|
| `Mod+P` | 窗口信息提取器 | 点选窗口后可复制标题、App ID、PID，或吸取屏幕颜色(HEX/RGB) |
| `Alt+F4` | 强制杀窗口 | 鼠标点选无响应窗口，强制结束进程 |
| `Alt+Shift+F4` | 强杀进程树 | 连同子进程一起结束（适用于自动重启的程序） |
| `Mod+Shift+P` | 电源菜单 | 锁屏、关屏、挂起、重启、关机 |

### 3.2 壁纸 & 主题

| 快捷键 | 功能 |
|--------|------|
| `Mod+Alt+W` | 手动选择壁纸 |
| `Mod+F10` | 随机切换壁纸 |
| `Mod+Shift+F10` | 随机下载动漫壁纸 |
| `Mod+Alt+M` / `Mod+Alt+T` | 切换 Matugen 颜色策略（9种配色方案可选） |

### 3.3 窗口切换

| 快捷键                         | 功能                        |
| --------------------------- | ------------------------- |
| `Alt+Tab` / `Alt+Shift+Tab` | 带缩略图的窗口切换器，列出**所有工作区**的窗口 |
| `Mod+Tab` / `Mod+Shift+Tab` | 带缩略图的窗口切换器，只列**当前工作区**的窗口 |
| `` Mod+` ``（反引号键）           | 切换同一应用的所有窗口               |

> 两点是我后改的：一是 scope——fork 的窗口切换器（`config.kdl` 里的 `recent-windows` 块）转换键可以设"当前工作区 / 当前显示器 / 全部工作区"三档，我让 `Alt+Tab` 看全部、`Mod+Tab` 看当前工作区；二是把脚本带的那套 fuzzel 文本列表切换器（原来绑在 `Alt+Tab` 上）弃用了，只留原生缩略图这一个。

### 3.4 快速聚焦（需安装 nirius）

| 快捷键 | 功能 |
|--------|------|
| `Mod+Shift+Q` | 快速聚焦到 QQ |
| `Mod+Shift+W` | 快速聚焦到微信 |
| `Mod+Shift+O` | 快速聚焦到 OpenCode |
| `Mod+Ctrl+G` | 切换工作区时自动跟随窗口 |

### 3.5 其他

| 快捷键 | 功能 |
|--------|------|
| `Mod+Shift+/` | 快捷键教程（fuzzy 搜索） |
| `Mod+Slash` | 临时浮动终端 |
| `Mod+Alt+O` | 打开 opencode |
| `Mod+Shift+Z` | 切换屏幕缩放效果 |
| `Mod+Alt+滚轮` | 放大/缩小屏幕 |
| `Mod+Ctrl+Shift+Alt+B` | 播放 Bad Apple |
| `Mod+V` | 剪贴板 |

---

## 4. 我自己后来配置的快捷键

> 以下快捷键不是脚本默认自带的，是我在使用中自己加进 `~/.config/niri/binds.kdl` 的。修改后需要用 `niri msg action reload-config` 重新加载配置。

### 4.1 联网工具 / 蓝牙工具（打开/关闭切换）

原本 Waybar 上的联网、蓝牙图标只能用鼠标点击。我给它们配了键盘快捷键，并且实现了"再按一次关闭"的切换逻辑：

```kdl
// 联网工具
Mod+Shift+N hotkey-overlay-title="联网工具 Network Manager" { spawn-sh "niri msg windows 2>/dev/null | grep -q 'App ID: \"network-tui\"' && niri msg action close-window || kitty --single-instance --class network-tui -e ~/.config/waybar/scripts/select-network-tui.sh"; }
// 蓝牙工具
Mod+Shift+B hotkey-overlay-title="蓝牙工具 Bluetooth" { spawn-sh "niri msg windows 2>/dev/null | grep -q 'App ID: \"bluetui\"' && niri msg action close-window || kitty --class bluetui -e bluetui"; }
```

原理（一行 shell 搞定，不用写脚本）：

1. `niri msg windows` 列出当前打开的窗口，用 `grep` 判断目标 App ID 的窗口是否已存在
2. 已存在 → 执行 `niri msg action close-window` 关闭它
3. 不存在 → 启动对应程序

> 提示：联网工具和 Waybar 图标点的是同一个东西——`select-network-tui.sh`，它会自动检测后端（iwd 就用 impala，wpa_supplicant 就用 nmtui）。
>
> 注意：`close-window` 关的是**当前聚焦**的窗口，想关哪个得先把焦点落到它上面。

### 4.2 程序启动器 Alt+Space（改过键位 + 再按关闭）

程序启动器（fuzzel）原本不在 `Alt+Space` 上，我改成了 `Alt+Space`，顺便加了个"再按一次关闭"：

```kdl
Alt+Space hotkey-overlay-title="程序菜单 Applauncher" { spawn-sh "pkill fuzzel || fuzzel"; }
```

原理：`pkill fuzzel || fuzzel`——开着就杀掉，没开就启动，一个键就是开关。

### 4.3 指令中心 Command Center

Waybar 中间有个图标（`custom/actions`），点击会弹出一个 Shorin 常用维护指令面板：快速存档/读档、更新镜像源、更新系统、系统清理、深度系统清理、联网工具、蓝牙工具。原来只能鼠标点，我给它配了 `Mod+Shift+C`，同样能"再按一次关闭"：

```kdl
// 后加的：指令中心（Command Center），打开/关闭常用维护指令面板
Mod+Shift+C hotkey-overlay-title="指令中心 Command Center" { spawn-sh "pkill fuzzel || ~/.config/waybar/scripts/command-center.sh"; }
```

原理和 4.2 的启动器同款：`pkill fuzzel || ...`，先把正在开的菜单毙掉，毙不掉说明没开，就走后面那句把它启动。

> 顺带一提，面板是动态的——不是 btrfs 就没有存/读档，没装蓝牙设备就没有蓝牙工具，选项按环境里有的命令自动增减。

---

## 5. Niri 窗口管理快捷键（Vim 键位）

### 5.1 切换焦点

| 快捷键 | 功能 |
|--------|------|
| `Mod+H` / `Mod+Left` | 左移焦点 |
| `Mod+J` / `Mod+Down` | 下移焦点 |
| `Mod+K` / `Mod+Up` | 上移焦点 |
| `Mod+L` / `Mod+Right` | 右移焦点 |
| `Mod+W` / `Mod+S` | 上/下切换窗口聚焦 |

### 5.2 移动窗口

| 快捷键 | 功能 |
|--------|------|
| `Mod+Ctrl+H` / `Mod+Ctrl+Left` | 窗口左移 |
| `Mod+Ctrl+J` / `Mod+Ctrl+Down` | 窗口下移 |
| `Mod+Ctrl+K` / `Mod+Ctrl+Up` | 窗口上移 |
| `Mod+Ctrl+L` / `Mod+Ctrl+Right` | 窗口右移 |
| `Mod+Ctrl+A` / `Mod+Ctrl+D` | 左/右移动列 |

### 5.3 列操作（窗口合并/踢出）

> 需要同一列中有多个窗口才有效果

| 快捷键 | 功能 |
|--------|------|
| `Mod+A` | 向左合并窗口到列 |
| `Mod+D` | 向右合并窗口到列 |
| `Mod+Shift+A` | 合并窗口到列 |
| `Mod+Shift+D` | 从列中踢出窗口 |
| `Mod+Comma`(逗号 ,) | 合并窗口到列 |
| `Mod+Period`(句号 .) | 从列中踢出窗口 |
| `Mod+X` | 标签页模式切换 |

### 5.4 工作区切换

| 快捷键 | 功能 |
|--------|------|
| `Mod+U` | 切换到下方工作区 |
| `Mod+I` | 切换到上方工作区 |
| `Mod+1` ~ `Mod+9` | 切换到指定工作区 |

### 5.5 移动窗口到工作区

| 快捷键 | 功能 |
|--------|------|
| `Mod+Ctrl+U` | 窗口移到下方工作区 |
| `Mod+Ctrl+I` | 窗口移到上方工作区 |
| `Mod+Ctrl+1` ~ `Mod+Ctrl+9` | 窗口移到指定工作区 |

### 5.6 窗口大小与布局

| 快捷键 | 功能 |
|--------|------|
| `Mod+R` | 按预设切换宽度 |
| `Mod+Shift+R` | 按预设切换高度 |
| `Mod+Ctrl+R` | 重置窗口高度 |
| `Mod+Minus` / `Mod+Equal` | 调整宽度 -5% / +5% |
| `Mod+Shift+Minus` / `Mod+Shift+Equal` | 调整高度 -5% / +5% |
| `Mod+C` | 居中当前列 |
| `Mod+F` | 最大化 |
| `Mod+Alt+F` | 全屏 |
| `Mod+M` | 最小化 |

### 5.7 浮动模式

| 快捷键               | 功能       |
| -------------------- | ---------- |
| `Mod+Shift+Space`    | 切换浮动/平铺 |

> 提示：浮动窗口按住 `Mod` + 鼠标左键拖动即可移动