#  Arch Linux 安装实战记录 (Btrfs + Zram)

> [!WARNING]
> **免责声明 (Disclaimer)**  
> 本文仅记录个人折腾 Arch Linux 的安装过程，供学习与交流使用。受硬件差异、软件版本迭代等因素影响，不同设备的执行结果可能会有出入。**折腾有风险，操作需谨慎！** 请务必在开始前**备份好重要数据**。如果跟着教程操作不慎导致数据丢失、把其他系统扬了或者硬件损坏，本人不仅概不负责, 还会狠狠地嘲笑你~ 

本文记录了个人安装 Arch Linux 的完整命令流程，采用了 `Btrfs` 文件系统并配置了子卷，最后启用了 `Zram` 优化。

## 1. 前置准备

### 1.1 终端显示与网络连接
安装初始阶段，建议调整字体大小以便于查看，并连接上 Wi-Fi 网络。

```bash
# 调整终端字体大小
setfont ter-v32n

# 使用 iwctl 连接 Wi-Fi
ip a
iwctl
device list
station wlan0 scan
station wlan0 get-networks
station wlan0 connect WIFI-Name
exit

# 测试网络连通性
ping bilibili.com
```

### 1.2 时间同步与镜像源配置
确保系统时间准确，并使用 `reflector` 挑选国内最快的镜像源，这能大幅缩短下载时间。

```bash
# 同步网络时间 (查看并确保 NTP service: active)
timedatectl 
# 若未开启, 可执行以下命令开启
# timedatectl set-ntp true

# 配置镜像源 (按速率排序，保留10个国内源)
reflector -a 12 -c cn -f 10 --sort rate --v --save /etc/pacman.d/mirrorlist
# 下面是的按照score排序
# reflector -a 12 -c cn -f 10 --sort score --v --save /etc/pacman.d/mirrorlist 

# 更新数据库并安装 Arch Linux 密钥环
pacman -Sy archlinux-keyring

# 安装 yazi 方便在终端中浏览文件
pacman -S yazi
```

---

## 2. 磁盘分区与文件系统

> [!CAUTION]
> **注意**：分区操作会清空数据，请仔细确认目标磁盘（以下使用 `/dev/{nvmexnx}` 代指目标盘）。

### 2.1 硬盘分区
首次使用硬盘，建议选择 `gpt` 分区表。
*   创建一个分区 `new` > `500MB`，类型选择 **EFI system**
*   剩余空间全部分给系统 `new` > 剩余空间，类型默认 **Linux filesystem**

```bash
# 查看当前分区情况
lsblk -pf

# 进入 cfdisk 进行可视化分区
cfdisk /dev/{nvmexnx}
```

### 2.2 格式化与 Btrfs 子卷创建
分区完成后，对 EFI 和系统分区进行格式化。为了方便后期快照管理，我们为根目录和 home 目录创建了独立的 Btrfs 子卷。

```bash
# 格式化 EFI 分区 (假设为 nvme0n1p1)
mkfs.fat -F 32 /dev/{nvme0n1p1}

# 格式化系统分区为 btrfs (假设为 nvme0n1p2)
mkfs.btrfs /dev/{nvme0n1p2}

# 临时挂载以创建子卷 (防止快照备份用户文件)
mount -t btrfs /dev/{nvme0n1p2} /mnt
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home

# 取消临时挂载
umount /mnt
```

### 2.3 挂载正式分区
开启 `zstd` 压缩以提升性能和节省空间。

```bash
# 挂载根目录子卷
mount -t btrfs -o subvol=/@,compress=zstd /dev/{nvme0n1p2} /mnt

# 挂载 home 目录子卷
mount --mkdir -t btrfs -o subvol=/@home,compress=zstd /dev/{nvme0n1p2} /mnt/home

# 挂载 EFI 分区
mount --mkdir /dev/{nvme0n1p1} /mnt/efi
```

---

## 3. 安装基础系统

使用 `pacstrap` 将基本系统和常用软件包安装到 `/mnt` 中。

```bash
# 安装基础系统、内核、微码、网络管理器及常用工具
pacstrap -K /mnt base base-devel linux-zen linux-firmware btrfs-progs networkmanager vim sudo intel-ucode

# 生成 fstab 文件，用于系统启动时自动挂载磁盘
genfstab -U /mnt > /mnt/etc/fstab
```

---

## 4. 系统基础配置

环境已经搭建好，现在切换到新系统中进行本地化和用户配置。

```bash
# chroot 切换进入新系统
arch-chroot /mnt 
```

### 4.1 时区与时间
```bash
# 设置时区为上海
timedatectl set-timezone Asia/Shanghai

# 将系统时间写入硬件时钟
hwclock --systohc
```
> [!NOTE]
> *(注：在 chroot 环境下也可以用 `ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime` 软链接的方式设置, 不过还是推荐使用set-timezone方式)*

### 4.2 本地化设置
```bash
# 1. 编辑 locale.gen
vim /etc/locale.gen
# 解除 `en_US.UTF-8 UTF-8` 和 `zh_CN.UTF-8 UTF-8` 前面的注释
# 如果是vim操作的话, 输入'/'进行搜索查找, 回车后光标定位, 按'x'剪切掉'#'即可

# 2. 生成本地化文件
locale-gen

# 3. 设置默认语言为英文（避免 tty 中文字符显示为方块）
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

### 4.3 主机名与 Root 密码
```bash
# 设置主机名 - 换成自己喜欢的名字即可, 终端的提示符会显示这个名称(user@hostname)
echo "pianone-arch" > /etc/hostname

# 设置 root 账户密码 (按提示输入两遍)
passwd
```

---

## 5. Bootloader (GRUB) 安装

我们使用 GRUB 来引导系统，并开启 `os-prober` 以支持多操作系统引导。

### 5.1 安装 GRUB 到 EFI
```bash
# 安装必要的软件包
pacman -S grub efibootmgr

# 安装 GRUB 引导程序 - bootloader-id默认为arch, 可以自定义, 作为grub启动选项的名称
grub-install --target=x86_64-efi --efi-directory=/efi --boot-directory=/efi --bootloader-id=Arch

# (可选) 如果没找到启动项，可尝试追加 --removable 参数
# grub-install --target=x86_64-efi --efi-directory=/efi --boot-directory=/efi --bootloader-id=arch --removable

# 创建 boot 到 efi 的连接，方便后续生成配置
ln -s /efi/grub /boot/grub
```

### 5.2 多系统支持配置
```bash
# 安装多系统检测工具
pacman -S os-prober exfat-utils

# 编辑 GRUB 配置文件
vim /etc/default/grub
```
在 `/etc/default/grub` 中修改/添加以下配置：
```ini
GRUB_DEFAULT=saved
GRUB_TIMEOUT=3
GRUB_SAVEDEFAULT=true
GRUB_DISABLE_OS_PROBER=false
```

---

## 6. Zram 内存压缩优化

为了提升系统响应速度并减少磁盘 Swap 的磨损，配置 Zram。

```bash
# 安装 zram 生成器
pacman -S zram-generator

# 配置 zram 参数
vim /etc/systemd/zram-generator.conf
```
写入以下配置：
```ini
[zram0]
zram-size = ram 
compression-algorithm = zstd
```

为了防止 zram 和内核自带的 zswap 冲突，我们需要在 GRUB 启动参数中禁用 zswap：
```bash
# 编辑 GRUB 配置文件
vim /etc/default/grub
# 修改 GRUB_CMDLINE_LINUX_DEFAULT，追加 zswap.enabled=0
# 例如：GRUB_CMDLINE_LINUX_DEFAULT='loglevel=5 zswap.enabled=0'

# 重新生成 GRUB 配置文件，使所有更改生效
grub-mkconfig -o /boot/grub/grub.cfg
```

---

## 7. 收尾工作

安装和配置已全部完成，退出 chroot 环境并重启！

```bash
# 切换用户后，可以在新系统中也装上 yazi
pacman -S yazi

# 退出 chroot
exit

# 重启系统
reboot
```
> [!TIP]
> 重启后进入 UEFI/BIOS，将启动首选项设置为 `Arch` 即可体验你的新系统啦！

---

## 8. 参考鸣谢

本文的安装流程主要参考了哔哩哔哩 [**林长枫Shorin709**](https://space.bilibili.com/9202840?spm_id_from=333.337) UP 主的优秀视频教程。如果你在操作过程中对某些细节仍有疑问，非常推荐配合原视频一起食用：

*   📺 **B站视频教程**：[【从LinuxMint入门到ArchLinux安装详解】(BV19DBqB4EY4)](https://www.bilibili.com/video/BV19DBqB4EY4/)
