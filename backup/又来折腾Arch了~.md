# Arch 安装命令执行记录

```bash
setfont ter-v32n

# 连接网络
ip a
iwctl
device list
station wlan0 scan
station wlan0 get-networks
station wlan0 connect WIFI-Name
exit

# 测试网络
ping bilibili.com

# 同步网络时间 查看并确保NTP service: actice
timedatectl
# 若未开启, 则执行下面的命令
# timedatectl set-ntp true

# 配置镜像源
# reflector -a 12 -c cn -f 10 --sort score --v --save /etc/pacman.d/mirrorlist
reflector -a 12 -c cn -f 10 --sort rate --v --save /etc/pacman.d/mirrorlist

# 更新数据库并安装密钥
pacman -Sy archlinux-keyring

# 安装 yazi 方便浏览文件
pacman -S yazi

# ======== 进行硬盘分区 ========
# 列出当前分区情况
lsblk -pf
# fdisk -l /dev/{nvmexnx}

# 查看目标分区情况
cfdisk /dev/{nvmexnx}
# 首次使用硬盘, 选择gpt分区进入
# 创建一个分区 new > 500MB 类型选择 EFI system
# 单硬盘剩余空间全部分给系统 new > 剩余空间 类型默认 Linux filesystem
# 退出cfdisk 并保存分区表

# 列出当前分区情况
lsblk -pf
# 格式化分区
# efi分区格式成fat -- 下面用{nvmexn1}代指efi分区
mkfs.fat -F 32 /dev/{nvmexn1}
# 系统格式成btrfs -- 下面用{nvmexn2}代指系统分区
mkfs.btrfs /dev/{nvmexn2}

# 创建子卷 (防止快照备份用户文件)
mount -t btrfs /dev/{nvmexn2} /mnt
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home

# 列出当前分区情况
lsblk -pf
# 取消当前挂载
umount /mnt
# 挂载子卷
mount -t btrfs -o subvol=/@,compress=zstd /dev/{nvmexn2} /mnt
mount --mkdir -t btrfs -o subvol=/@home,compress=zstd /dev/{nvmexn2} /mnt/home
mount --mkdir /dev/{nvmexn2} /mnt/efi

# 正式安装系统
pacstrap -K /mnt base base-devel linux linux-firmware btrfs-progs
pacstrap /mnt networkmanager vim sudo intel-ucode iwd

# 生成fstab文件 用于系统启动时挂载
genfstab -U /mnt > /mnt/etc/fstab

# 切换用户
arch-chroot /mnt 





```