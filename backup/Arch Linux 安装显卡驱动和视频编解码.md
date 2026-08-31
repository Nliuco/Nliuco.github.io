
> [!WARNING]
> **免责声明 (Disclaimer)**  
> 本文仅记录个人折腾 Arch Linux 的显卡驱动、视频编解码与音频服务配置过程，供学习与交流使用。受硬件差异（尤其是各具特色的双显卡笔记本架构）、驱动分支迭代等因素影响，不同设备的执行结果可能会有出入。**折腾有风险，回车需谨慎！** 请务必在操作前**做好系统快照备份**。如果跟着教程操作不慎导致黑屏进不去系统、驱动滚挂或者把硬件干冒烟了，本人不仅概不负责，还会狠狠地嘲笑你~

本文记录了 Arch Linux 在基础系统配置完成后的显卡驱动、视频硬件编解码、PipeWire 音频服务以及蓝牙与常用桌面基础组件的配置全流程。针对 Intel 核显 + NVIDIA 独显（Optimus）笔记本架构，提供了从驱动型号识别、DKMS 编译到硬件加速验证的完整实践记录。

---

## 1. 前置准备与系统更新

在安装任何硬件驱动之前，建议先将整个系统更新至最新状态，并建立基准快照。这样即使后续驱动编译失败或配置出错，也可以随时一键回滚。

```bash
# 全系统升级（确保内核与软件版本最新）
pacman -Syu

# 在安装显卡驱动前创建 Snapper 快照备份
sudo snapper -c root create --description "before graphics driver"
sudo snapper -c home create --description "before graphics driver"
```

---

## 2. NVIDIA 独显驱动配置 (Optimus 架构)

对于配备 Intel 核显 + NVIDIA 独显（Optimus 架构）的笔记本，日常推荐使用核显负责显示输出以节省电量，在需要运行大型软件或游戏时再通过 PRIME 调用独显。

### 2.1 安装内核头文件
由于后续需要使用 DKMS（Dynamic Kernel Module Support）在内核更新时自动重新编译 NVIDIA 模块，必须先安装当前所有在用内核对应的 Headers。

```bash
# 安装已安装内核的头文件（以 linux-zen 和备用 linux-lts 为例）
pacman -S --needed linux-zen-headers linux-lts-headers
```

### 2.2 确认显卡芯片代号与驱动分支
NVIDIA 驱动版本繁多，必须先准确确认自己显卡的芯片代号（Family）。

```bash
# 查看系统中的显卡硬件设备
lspci -nn | grep -E 'VGA|3D'

# 或者使用官方推荐的查询命令
lspci -k -d ::03xx
```

> [!TIP]
> **如何根据硬件确定驱动版本：**  
> 1. 执行上述命令后，可以看到独显的型号和代号。例如显卡显示为 `GeForce MX250`，前面的芯片代号为 `GP108BM`。
> 2. 查阅 [Nouveau Wiki CodeNames 页面](https://nouveau.freedesktop.org/CodeNames.html)，搜索 `GP108` 可知其属于 **NV130 family (Pascal 架构)**。
> 3. 查阅 [Arch Linux NVIDIA Wiki](https://wiki.archlinux.org/title/NVIDIA) 的 *Installation* 章节，找到 Pascal 架构对应的驱动分支（例如本文适用的 `nvidia-580xx-dkms` 或最新官方对应分支）。

### 2.3 安装显卡驱动与 PRIME 工具
确定好对应的驱动分支后，使用 AUR 助手安装 DKMS 驱动包、用户态工具以及显卡切换工具 `nvidia-prime`。

```bash
# 安装 NVIDIA 驱动、用户空间工具与 PRIME 切换脚本
yay -S nvidia-580xx-dkms nvidia-580xx-utils nvidia-prime
```
*   `nvidia-580xx-dkms`：针对不同内核自动编译内核模块；
*   `nvidia-580xx-utils`：提供 NVIDIA 控制工具与用户空间依赖；
*   `nvidia-prime`：提供 `prime-run` 包装命令，实现按需使用独显启动程序。

### 2.4 驱动状态检查与生效验证
安装完毕后，在重启前先检查 DKMS 模块编译状态：

```bash
# 检查 DKMS 模块是否已成功针对所有内核编译并安装 (应显示 installed)
sudo dkms status

# 检查相关软件包是否安装完整
pacman -Q | grep -E 'nvidia|linux-(zen|lts).*headers'

# 查看当前显卡占用的内核模块（此时通常仍为开源的 nouveau）
lspci -k -s 01:00.0
```

确认无误后重启系统：
```bash
reboot
```

重启进入系统后，验证 NVIDIA 专有驱动是否成功接管：
```bash
# 预期输出应包含：Kernel driver in use: nvidia
lspci -k -s 01:00.0

# 查看 NVIDIA 显卡运行状态与驱动信息
nvidia-smi
```

### 2.5 Vulkan 与 OpenGL 验证
安装常用的图形验证工具包：

```bash
# 安装 Vulkan 与 OpenGL 常用调试和信息查看工具
sudo pacman -S vulkan-tools mesa-utils
```

```bash
# 在纯 TTY 终端下即可验证 Vulkan 设备识别情况
vulkaninfo --summary | grep -E 'GPU|deviceName'
```

> [!NOTE]
> **关于 OpenGL 验证的说明**  
> `glxinfo` 依赖 X11 或 Wayland 的图形会话，如果在纯 TTY 控制台环境下运行会提示 `Error: unable to open display`，这属于正常现象。待后续进入桌面环境后，即可执行以下命令进行双显卡验证：
> 
> ```bash
> # 预期输出：Intel UHD Graphics 620（默认使用低功耗核显）
> glxinfo -B | grep -E 'OpenGL renderer|OpenGL version'
> 
> # 预期输出：NVIDIA GeForce MX250（成功调用独显渲染）
> prime-run glxinfo -B | grep -E 'OpenGL renderer|OpenGL version'
> ```

---

## 3. Intel 核显与视频硬件编解码 (VA-API / FFmpeg)

为了在观看视频或运行多媒体应用时大幅降低 CPU 占用并提升续航，需要配置 Intel 核显的 VA-API 硬件编解码支持。

### 3.1 阶段快照备份
```bash
# 在安装编解码组件前创建快照
sudo snapper -c root create --description "before video acceleration"
sudo snapper -c home create --description "before video acceleration"
```

### 3.2 安装媒体驱动与编解码包
```bash
# 安装 Intel 媒体驱动、VA-API 验证工具以及 FFmpeg
sudo pacman -S intel-media-driver libva-utils ffmpeg
```

> [!IMPORTANT]
> **FFmpeg 依赖选择提示**  
> 在安装 `ffmpeg` 过程中，包管理器可能会提示选择 JACK 的 Provider：
> ```text
> :: There are 2 providers available for Jack:
>    1) jack2  2) pipewire-jack
> ```
> 如果后续计划使用现代化的 **PipeWire** 作为整套系统的音频中枢（并搭配如 niri、Hyprland 等 Wayland 桌面），**强烈推荐选择 `2) pipewire-jack`**，让 JACK 应用程序统一走 PipeWire 处理。

### 3.3 验证硬件编解码能力
```bash
# 1. 检查 Direct Rendering Infrastructure (DRI) 节点
ls /dev/dri
# 正常应能看到 card0 以及用于 GPU 渲染硬件加速的 renderD128

# 2. 检查 Intel VA-API 驱动支持的硬件编解码配置
LIBVA_DRIVER_NAME=iHD vainfo --display drm --device /dev/dri/renderD128
```
若输出中包含 `VAProfileH264Main`、`VAProfileH264High`、`VAProfileHEVCMain`、`VAProfileVP9Profile0` 等行且后面标有 `VAEntrypointVLD`（解密/解码支持），说明 Intel 核显的硬件编解码驱动已正常就绪。

```bash
# 3. 查看 FFmpeg 编译支持的硬件加速后端
ffmpeg -hwaccels
```
> [!NOTE]
> `ffmpeg -hwaccels` 输出中如果包含 `vaapi`、`vulkan` 等项，即代表已具备相应的加速调用支持。

---

## 4. PipeWire 音频系统与蓝牙配置

在现代 Linux 生态中，推荐使用 **PipeWire** 全面替代传统的 ALSA/PulseAudio/JACK 方案，提供超低延迟与统一的音视频路由管理。

### 4.1 阶段快照备份
```bash
# 在安装音频和蓝牙服务前创建快照
sudo snapper -c root create --description "before audio video bluetooth services"
sudo snapper -c home create --description "before audio video bluetooth services"
```

### 4.2 安装底层音频固件与 ALSA 配置
```bash
# 安装 Intel 现代音频设备必备固件与 ALSA 配置
sudo pacman -S sof-firmware alsa-ucm-conf

# (可选) 若设备较老且出现未识别声卡情况，可补装传统固件
# sudo pacman -S alsa-firmware
```

### 4.3 安装现代化 PipeWire 音频栈
```bash
# 安装 PipeWire 完整音频栈与会话管理器
sudo pacman -S pipewire pipewire-audio pipewire-alsa pipewire-pulse pipewire-jack wireplumber
```
组件分工简述：
*   `pipewire`：核心媒体处理守护进程；
*   `pipewire-audio`：核心音频处理模块；
*   `pipewire-alsa`：为传统 ALSA 程序提供兼容层；
*   `pipewire-pulse`：为 PulseAudio 客户端提供无缝替换的守护服务；
*   `pipewire-jack`：为专业音频 JACK 客户端提供低延迟兼容层；
*   `wireplumber`：现代化的 PipeWire 会话与策略管理器。

### 4.4 启用用户音频服务与系统蓝牙
```bash
# 启动并启用当前用户的 PipeWire 音频服务（用户级服务，不需要 sudo）
systemctl --user enable --now pipewire pipewire-pulse wireplumber

# 安装并启动蓝牙服务（系统级服务，需要 sudo）
sudo pacman -S bluez bluez-utils
sudo systemctl enable --now bluetooth
```

### 4.5 音视频与蓝牙状态排查指南
执行以下命令检查各服务及设备的运行健康度：

```bash
# 检查当前用户音频服务运行状态
systemctl --user is-active pipewire pipewire-pulse wireplumber

# 检查音频拓扑节点与设备输出
wpctl status

# 检查 PulseAudio 兼容接口是否成功桥接到 PipeWire
pactl info

# 检查蓝牙控制器状态
bluetoothctl show

# 检查无线设备是否被硬件/软件射频屏蔽
rfkill list
```

**预期状态对照表：**

| 检查项目 | 执行命令 | 正常状态 (Expected) | 异常状态 (Issues) |
| :--- | :--- | :--- | :--- |
| **PipeWire 核心** | `systemctl --user is-active pipewire` | `active` | `inactive` 或 `failed` |
| **PulseAudio 兼容层** | `systemctl --user is-active pipewire-pulse` | `active` | `inactive` 或 `failed` |
| **WirePlumber 会话管理** | `systemctl --user is-active wireplumber` | `active` | `inactive` 或 `failed` |
| **音频设备识别** | `wpctl status` | 能看到 `Audio` 下的声卡、`Sinks`（输出）及 `Sources`（输入） | 没有 `Audio` 分组或设备列表为空 |
| **Pulse 桥接状态** | `pactl info` | 输出包含 `Server Name: PulseAudio (on PipeWire ...)` | 提示 `Connection failure` 无法连接 |
| **蓝牙系统服务** | `systemctl is-active bluetooth` | `active` | `inactive` 或 `failed` |
| **蓝牙硬件控制器** | `bluetoothctl show` | 看到 `Controller ...` 且包含 `Powered: yes` | 提示 `No default controller available` |
| **无线设备射频屏蔽** | `rfkill list` | Bluetooth 项中 `Soft/Hard blocked` 均为 `no` | 出现 `blocked: yes`（需执行 `rfkill unblock bluetooth`） |

```bash
# 服务配置完成，打上快照
sudo snapper -c root create --description "after audio video bluetooth services"
sudo snapper -c home create --description "after audio video bluetooth services"
```

---

## 5. 常用基础组件与优化杂项 (可选)

在正式安装桌面环境（如 KDE / GNOME / Niri / Hyprland）之前，建议先将电源调度、中文与等宽字体以及通用应用分发格式（Flatpak）配置到位。

### 5.1 电源管理服务
```bash
# 安装电源模式管理工具（支持平衡、节能与高性能模式切换）
sudo pacman -S power-profiles-daemon
sudo systemctl enable --now power-profiles-daemon
```

### 5.2 基础中文字体与 Emoji 支持
为了防止后续进入桌面环境或打开网页时中文字符变成“豆腐块”乱码，预先安装基础字族：

```bash
# 安装 Noto 基础字族、Emoji 表情字体及 Adobe 思源黑体
sudo pacman -S noto-fonts noto-fonts-emoji adobe-source-han-sans-cn-fonts

# (推荐) 程序员必备的 JetBrains Mono 与 Nerd 图标字体
# sudo pacman -S ttf-jetbrains-mono ttf-jetbrains-mono-nerd
```

### 5.3 Flatpak 软件包管理器与国内源配置
有些扩展比较多的软件，Flatpak版本通常比AUR上的更好用, 比如OBS和Easyeffects.
对于部分依赖复杂或未进官方源的应用，也可以使用 Flatpak 安装并配置国内镜像加速：

```bash
# 安装 Flatpak
sudo pacman -S flatpak

# 查看当前 remotes 并配置上海交通大学 Flathub 镜像源
flatpak remotes
sudo flatpak remote-modify flathub --url=https://mirror.sjtu.edu.cn/flathub
```

---

## 6. 收尾工作与环境就绪

到这里，Arch Linux 的双显卡驱动、硬件视频编解码、音频中枢、蓝牙通信及基础字体全部配置完毕！

```bash
# 为当前整洁完整的驱动与多媒体环境创建快照
sudo snapper -c root create --description "before desktop environment finally"
sudo snapper -c home create --description "before desktop environment finally"

# 重启电脑
reboot
```

> [!TIP]
> 搞到这里，双显卡驱动、硬件编解码、音频和蓝牙服务就都已经稳稳当当了，更重要的是一路打下来的 Snapper 快照让我们随时有了“后悔药”。
> 
> 底层基础全部搭好，接下来终于可以放开手脚折腾图形界面了（后续打算搞一下 Niri + Noctalia 这套配置），咱们下篇见！
