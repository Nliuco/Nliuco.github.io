
> [!NOTE]
> **免责声明 (Disclaimer)**  
> 本文仅为个人 Arch Linux 基础配置记录，供学习参考。受软件迭代影响，部分操作可能随时间变化。**折腾有风险，回车需谨慎！** 若因盲目复制粘贴导致系统“滚挂”或配置错乱，本人不仅概不负责，还会狠狠地嘲笑你。

本文记录了 Arch Linux 基础安装完成后的系统配置流程，包含网络设置、用户管理、软件源添加以及非常关键的 Btrfs 快照（Snapper）配置。

## 1. 基础网络与系统更新

刚安装好的新系统，需要先配置网络并进行全系统升级。

```bash
# 开启 NetworkManager 开机自启服务
systemctl enable --now NetworkManager

# 运行 nmtui 打开终端网络配置界面
nmtui

# 连接网络后退出，测试连接情况
ping bilibili.com

# 安装 fastfetch 用于查看系统信息
pacman -S fastfetch

# 更新系统 (小写 u 代表升级所有软件)
pacman -Syu
```

## 2. 环境变量与系统默认设置

将默认编辑器设置为 `vim`。

```bash
# 将 EDITOR=vim 追加写入到环境变量文件中
echo "EDITOR=vim" >> /etc/environment
exit
```
完成后执行 `exit` 退出并重新登录 root 账户使环境变量生效。

## 3. 用户与权限管理

日常使用不建议直接使用 root 账户，我们需要创建一个拥有管理员权限的普通用户。

```bash
# 创建普通用户 
# -m: 创建用户时同时创建 home 目录
# -G wheel: 添加到管理员权限组(wheel)
useradd -mG wheel pianone

# 设置用户密码
passwd pianone

# 运行 visudo 设置管理员权限
visudo
```
在 `visudo` 编辑界面中，使用 `/wheel` 搜索，找到如下行，按 `x` 删除开头的 `#` 号取消注释：
```ini
%wheel ALL=(ALL:ALL) ALL
```

## 4. ArchlinuxCN 源与 AUR 助手

为了更方便地安装软件，我们需要开启 32 位软件源并添加国内著名的 ArchlinuxCN 源。

```bash
# 编辑 pacman 配置文件
vim /etc/pacman.conf
```
在文件中进行以下两步操作：
1. 找到 `[multilib]`，取消它和下一行 `Include` 的注释。
2. 在文件末尾追加 ArchlinuxCN 源配置：

```ini
[multilib]
Include = /etc/pacman.d/mirrorlist

# 在文件末尾添加以下内容
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
```

保存退出后，同步数据并安装密钥环和 AUR 助手：
```bash
# 同步数据并安装 ArchlinuxCN 的密钥环
pacman -Sy archlinuxcn-keyring

# 配置好后，安装 AUR 助手
pacman -S yay paru
```

## 5. Btrfs 快照管理 (Snapper)

为了防止系统被“滚挂”，配置 Snapper 快照是 Arch 用户的必备技能。

### 5.1 安装依赖与初始化快照
```bash
# 安装 snapper 及相关辅助工具
pacman -S snapper snap-pac btrfs-assistant grub-btrfs inotify-tools

# 开启 grub-btrfs 服务开机自启
systemctl enable --now grub-btrfsd

# 重启电脑，让 pacman 自动创建快照功能生效
reboot
```

重启并重新登录 root 账户后，初始化 Snapper 配置文件：
```bash
# 为 root 和 home 目录创建配置文件
snapper -c root create-config /
snapper -c home create-config /home

# 创建“初始纯净快照” (快照 #1)
snapper -c root create --description "init arch snapshot"
snapper -c home create --description "init arch snapshot"
```

### 5.2 LTS 内核与快照更新
安装 LTS 内核作为备用，并更新 GRUB 启动项，这样可以在 GRUB 菜单中直接选择进入快照。
```bash
# 安装 LTS 内核以备不时之需
pacman -S linux-lts

# 更新 GRUB 配置文件，生成 LTS 内核启动项和快照启动项
grub-mkconfig -o /boot/grub/grub.cfg

# 为“安装了 LTS 内核”的状态再创建一个快照 (快照 #2)
snapper -c root create --description "with lts kernel and grub updated"
```

## 6. 快照恢复操作指南

当系统出现问题时，你可以使用命令行或图形化工具进行回档。

> [!WARNING]  
> **注意**：处理快照时最好以 `root` 身份执行。Snapper官方文档强烈建议：**请不要在使用 root 文件系统时使用 Snapper `undochange` 命令**，因为这样做可能会导致系统故障。推荐使用 Btrfs 助手或在 Live CD 中进行恢复。

```bash
# 命令行列出所有可用快照
snapper -c root list

# 使用 btrfs 助手命令行工具查看
btrfs-assistant -l

# 使用助手进行快照恢复 (用数字序号 n 指定要使用的快照)
btrfs-assistant -r n

# 删除快照
# snapper -c root delete n
```

---

## 7. 拓展知识：Snapper undochange 高级用法

虽然不建议在活动的根目录使用 `undochange`，但了解它的逻辑非常有用。`undochange n1..n2` 代表**选择性撤销**，而不是“一刀切”地恢复整个系统。

> [!NOTE]  
> **假设场景：**  
>
> - **快照1**（昨天上午）：系统正常。  
> - **快照2**（昨天下午）：误删文件 `A`。  
> - **快照3**（今天上午）：创建新文件 `B`，修改文件 `C`。  
>
> 此时你只想找回文件 `A`，但要保留今天的文件 `B` 和 `C`。
> - ❌ `undochange 1..0`：撤销从快照1至今的所有变动，今天的文件 `B` 也会丢失。
> - ✅ `undochange 1..2`：仅计算快照1和快照2之间的变化（即文件 `A` 被删），并**仅撤销这部分变化**。快照3之后创建的 `B` 和 `C` 纹丝不动。

`undochange` 的执行逻辑是：
1. 查看**起点编号**的文件状态。
2. 查看**终点编号**的文件状态。
3. 计算从起点到终点**发生了哪些具体的增删改**。
4. 在当前系统中，把这些变化**反过来执行一遍**。

因此，`n1..n2` 指代的是 **“从第一个快照到第二个快照之间的那个时间段”**，而不是两个“目标位置”。
