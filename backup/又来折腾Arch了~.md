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



```