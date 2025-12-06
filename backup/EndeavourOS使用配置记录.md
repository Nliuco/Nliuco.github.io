下文中涉及到的部分文档: [Arch Wiki Fcitx5](https://wiki.archlinux.org/title/Fcitx5)、[Arch Wiki Rime](https://wiki.archlinux.org/title/Rime)、[Arch Wiki Zram](https://wiki.archlinux.org/title/Zram)、[Arch Wiki KDE_Wallet](https://wiki.archlinux.org/title/KDE_Wallet#Disable_KWallet)、[clash-verge-rev](https://www.clashverge.dev/install.html#__tabbed_2_3)


## 服务不可用

### 1. paru: 未找到命令
```bash
yay -S paru
```

### 2. 蓝牙无法正常打开
- 先查看蓝牙服务是否正常启用, 尝试使用`systemctl`重新启用, 亦或是需要安装额外包
```bash
systemctl status bluetooth.service # 状态
sudo pacman -S bluez bluez-utils
sudo systemctl enable --now bluetooth.service
systemctl status bluetooth
```

### 3. 输入法不自带中文输入
- 使用`Fcitx5`输入框架
```bash
sudo pacman -S fcitx5-im fcitx5-chinese-addons
paru -S fcitx5-skin-ori-git # 安装皮肤-可选
```
- 为了使其他应用内部正确使用`Fcitx5`, 需要配置一下`/etc/environment`
```bash
sudo nano /etc/environment
```
- 添加以下内容
```
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
```
> [!NOTE]
> 可选: 安装`Rime`, 根据个人使用习惯,  配置`小鹤双拼`输入. 这里采用[雾凇拼音](https://github.com/iDvel/rime-ice) 
```bash
# 安装rime
sudo pacman -S fcitx5-rime
# 雾凇拼音方案
paru -S rime-ice-git
```
- 进入`~/.local/share/fcitx5/rime/`目录，创建配置`default.custom.yaml`，配置输入方案
```bash
cd ~/.local/share/fcitx5/rime/
nano default.custom.yaml
```
```yaml
patch:
  # 仅使用「雾凇拼音」的默认配置，配置此行即可
  __include: rime_ice_suggestion:/
  # 以下根据自己所需自行定义，仅做参考。
  # 针对对应处方的定制条目，请使用 <recipe>.custom.yaml 中配置，例如 rime_ice.custom.yaml
  __patch:
    key_binder/bindings/+:
      # 开启逗号句号翻页
      - { when: paging, accept: comma, send: Page_Up }
      - { when: has_menu, accept: period, send: Page_Down }
```
- 重启后刷新配置
```bash
fcitx5-remote -r
reboot # 没生效就重启一下系统, 或者点击一些托盘应用的重新启动Fcitx5.
```
### 4. 检查`NVIDIA`显卡驱动
- 查看是否检测到 NVIDIA 显卡
```bash
lspci -k | grep -A 3 -E "VGA|3D"
lspci | grep "NVIDIA"
```
- 输出中如果看到：
```bash
Kernel driver in use: nvidia
```
- 说明驱动 已加载成功。如果显示：
```bash
Kernel driver in use: nouveau
```
- 那就是 开源驱动（nouveau）被启用 → NVIDIA 专有驱动没正常加载。需要按照下面的步骤安装、配置
```bash
sudo pacman -S nvidia nvidia-utils nvidia-settings 
sudo pacman -S nvidia-prime # 混合显卡, 即cpu集显 + gpu独集显需要
# 再次验证
lspci -k | grep -A 3 -E "VGA|3D"
nvidia-smi
prime-run glxinfo | grep "OpenGL renderer" 
prime-run firefox # 这里用firefox浏览器测试一下独显是否使用正常
nvidia-smi # 再打开一个终端, 查看`Processes`下是否有firefox,  GPU现存占用情况等信息
```
### 5. kde wallet service 频繁弹出
- 编辑`~/.config/kwalletrc`文件
```
[Wallet]
Enabled=false
```

## 常用软件包
```bash
sudo pacman -Sy fastfetch
sudo pacman -S timeshift
sudo pacman -S clash-verge-rev
sudo pacman -S deskflow
sudo pacman -S wl-clipboard
sudo pacman -S fuse2 # 正常运行AppImage软件所需库
yay -S localsend-bin
paru -S bibata-cursor-theme-bin # 光标主题
yay -S wemeet-bin # 腾讯会议
sudo pacman -Rns wemeet-bin # 卸载腾讯会议
sudo pacman -Syu steam # 游戏
---
# 字体
sudo pacman -Sy noto-fonts-cjk noto-fonts-emoji tty-dejavu tty-jetbrains-mono-nerd
```

## 其他
### 0. 启用`ufw`防火墙
```bash
# 查看 UFW 当前状态（是否启用、默认策略、已开放端口等详细信息）
sudo ufw status verbose
# 没有安装的话就去安装 UFW
sudo pacman -S ufw
# 启用 UFW，并加载默认规则（防火墙开始生效）
sudo ufw enable
# 启动 ufw systemd 服务，并设置开机自动启动
sudo systemctl enable ufw --now
# 将默认入站策略设置为 "拒绝所有外部进入连接"，提高安全性
sudo ufw default deny incoming
# 默认允许所有出站流量（如浏览器上网、软件更新）避免影响正常使用
sudo ufw default allow outgoing
```
### 1. git拉取慢
- 配置本地代理服务器, 我这里使用的[clash-verge-rev](https://www.clashverge.dev/install.html#__tabbed_2_3)
```bash
git config --global http.proxy http://127.0.0.1:7897
git config --global https.proxy http://127.0.0.1:7897
```
### 2. localsend 无法被发现
- 其他设备无法检测到本机的localsend服务, 需要配置本地防火墙，开放`TCP/UDP`端口
```bash
sudo ufw allow 53317/tcp
sudo ufw allow 53317/udp
sudo ufw reload
```
### 3. SSD 启用定期 TRIM
- 这会告诉 SSD：
  - 哪些数据块已经被删除
  - 可以回收、擦除、重新整理
- 这能让 SSD:
  - 保持最高写入速度
  - 延长使用寿命
  - 减少随机写放大
- 每周自动执行一次 TRIM（对 SSD 清理无效块）
```bash
systemctl status fstrim.timer
sudo systemctl enable fstrim.timer --now
```
### 4. 使用 Zram
- Zram 是把内存的一部分压缩起来，当作“压缩内存交换区”来使用，提高系统的流畅性。
- 比传统 Swap 快很多, 也能减少 SSD 磨损 -> 因为少写入硬盘了。
- 1. 安装
```bash
sudo pacman -Sy zram-generator
```
- 2. 进行适当配置
```bash
sudo nano /etc/systemd/zram-generator.conf
```
- 3. 添加以下内容, 见[文档](https://github.com/systemd/zram-generator/blob/main/man/zram-generator.conf.md)
```conf
[zram0]
zram-size = ram / 2
compression-algorithm = zstd
```
- 4. 重启生效
```bash
sudo systemctl daemon-reload
sudo reboot
```
- 5. 验证
```bash
lsblk # 输出类似zram0   253:0   0   12G   0 disk [SWAP]
swapon --show # 输出类似 /dev/zram0 partition 12G    0B  100
```
- 6. 若`swapon --show`后系统还是在使用`swap`交换文件, 可以考虑禁用掉, 只使用zram
```bash
sudo swapoff /swapfile
sudo nano /etc/fstab
```
- 7. 注释掉以下行
```
# /swapfile none swap defaults 0 0
```
- 8. 再次验证
```bash
swapon --show
```
