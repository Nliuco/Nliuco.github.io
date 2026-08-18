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


```