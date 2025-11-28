下文中涉及到的部分文档: [Arch Wiki Fcitx5](https://wiki.archlinux.org/title/Fcitx5)、[Arch Wiki Rime](https://wiki.archlinux.org/title/Rime)、[clash-verge-rev](https://www.clashverge.dev/install.html#__tabbed_2_3)

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
- 重启rime加载刷新配置
```bash
fcitx5-remote -r
reboot # 没生效就重启一下系统, 或者点击一些托盘应用的重新启动Fcitx5.
```
## 常用软件包
```bash
sudo pacman -S timeshift
sudo pacman -S clash-verge-rev
sudo pacman -S wl-clipboard
sudo pacman -S fuse2 # 正常运行AppImage软件所需库
paru -S bibata-cursor-theme-bin # 光标主题
```

## 其他
### 1. git拉取慢
- 配置本地代理服务器, 我这里使用的[clash-verge-rev](https://www.clashverge.dev/install.html#__tabbed_2_3)
```bash
git config --global http.proxy http://127.0.0.1:7897
git config --global https.proxy http://127.0.0.1:7897
```

