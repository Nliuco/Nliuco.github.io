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
# efi分区格式成fat -- 下面用{nvme0n1p1}代指efi分区
mkfs.fat -F 32 /dev/{nvme0n1p1}
# 系统格式成btrfs -- 下面用{nvme0n1p2}代指系统分区
mkfs.btrfs /dev/{nvme0n1p2}

# 创建子卷 (防止快照备份用户文件)
mount -t btrfs /dev/{nvme0n1p2} /mnt
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home

# 列出当前分区情况
lsblk -pf
# 取消当前挂载
umount /mnt
# 挂载子卷
mount -t btrfs -o subvol=/@,compress=zstd /dev/{nvme0n1p2} /mnt
mount --mkdir -t btrfs -o subvol=/@home,compress=zstd /dev/{nvme0n1p2} /mnt/home
mount --mkdir /dev/{nvme0n1p1} /mnt/efi

# 正式安装系统
pacstrap -K /mnt base base-devel linux linux-firmware btrfs-progs
# pacstrap -K /mnt base base-devel linux linux-firmware btrfs-progs linux-firmware-marvell
pacstrap /mnt networkmanager vim sudo intel-ucode iwd

# 生成fstab文件 用于系统启动时挂载
genfstab -U /mnt > /mnt/etc/fstab

# 切换用户
arch-chroot /mnt 

# 设置时区软连接, 也可以用下面的设置时区的方式
# ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

# 切换用户后, 在系统里面再次安装yazi
pacman -S yazi

# 设置时区
timedatectl set-timezone Asia/Shanghai
# 查看时间是否正确
timedatectl
# 调整时间误差
hwclock --systohc

# 本地化语言设置
vim /etc/locale.gen
# 解除 `en_US.UTF-8 UTF-8` 和 `zh_CN.UTF-8 UTF-8` 前面的注释
# 生成本地化文件
locale-gen
# 编辑conf文件设置本地化
vim /etc/locale.conf
# 写入 LANG=en_US.UTF-8

# 设置主机名
vim /etc/hostname
# 可以写自己喜欢的名字, 这里我写了 pianone-arch

# 设置root账户密码
passwd
# 重复输入两遍密码

# 安装 Bootloader 引导加载程序
pacman -S grub efibootmgr

# 使用 grub-install 命令安装引导 - bootloader-id可以不指定, 默认是arch, 也可以换成别的
grub-install --target=x86_64-efi --efi-directory=/efi --boot-directory=/efi --bootloader-id=arch

# 用yazi看一眼/efi目录, 正常会有EFI和grub两个文件夹, 都正常有文件
# 且EFI/arch(上面配置的bootloader-id名称)下生成了grubx64.efi文件
yazi /efi
# 如果没找到arch的启动项, 可以通过追加下面的参数尝试解决
# grub-install --target=x86_64-efi --efi-directory=/efi --boot-directory=/efi --bootloader-id=arch --removable

# 创建 boot grub -> efi grub 的连接
ln -s /efi/grub /boot/grub

# 生成grub配置文件
grub-mkconfig -o /boot/grub/grub.cfg
# 如果没有 /efi/grub，创建它：
# mkdir -p /efi/grub

# 多操作系统识别
pacman -S os-prober exfat-utils
os-prober 

# 编辑grub文件
vim /etc/default/grub
# 配置以下条目
GRUB_DEFAULT=saved
GRUB_TIMEOUT=3
GRUB_SAVEDEFAULT=true
GRUB_DISABLE_OS_PROBER=false

# 再生成grub配置文件
grub-mkconfig -o /boot/grub/grub.cfg

# 配置zram
pacman -S zram-generator
# 配置zram
vim /etc/systemd/zram-generator.conf
# 配置信息如下
[zram0]
zram-size = ram 
compression-algorithm = zstd
# 编辑grub文件
vim /etc/default/grub
# 配置以下条目
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet zswap.enabled=0"

# 再生成grub配置文件
grub-mkconfig -o /boot/grub/grub.cfg

# 退出并重启
exit
reboot

# 进入UEFI/BIOS
# 设置启动首选项为arch

```