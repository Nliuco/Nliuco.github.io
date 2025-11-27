# EndeavourOS使用过程中遇到的问题
## 0. git拉取慢
- 配置本地代理服务器, 我这里使用的clash-verge-rev-bin
```bash
git config --global http.proxy http://127.0.0.1:7897
git config --global https.proxy http://127.0.0.1:7897
```

## 服务不可用
### 1. 蓝牙无法正常打开
- 先查看蓝牙服务是否正常启用, 尝试使用`systemctl`重新启用, 亦或是需要安装额外包
```bash
systemctl status bluetooth.service # 状态
sudo pacman -S bluez bluez-utils
sudo systemctl enable --now bluetooth.service
systemctl status bluetooth
```

### 2. paru: 未找到命令
```bash
yay -S paru
```

### 3. 输入法不自带中文输入
- 使用Fcitx5输入框架, 安装Rime, 根据个人使用习惯,  配置小鹤双拼输入
> [!NOTE]
> 采用[凇鹤拼音](https://github.com/kchen0x/rime-crane) —— Rime 简体中文输入法方案，整合了雾凇拼音和小鹤双拼/音形方案的拼音输入法。
```bash
sudo pacman -S fcitx5-im fcitx5-chinese-addons fcitx5-skin-ori-git
# 创建统一保存 git 仓库的目录, 后续可以将不变动位置的项目都放到这里面
mkdir -p ~/.local/share/repos
git clone https://github.com/kchen0x/rime-crane.git ~/.local/share/repos/rime-crane
rm -rf ~/.local/share/fcitx5/rime && ln -sif ~/.local/share/repos/rime-crane ~/.local/share/fcitx5/rime
# 编辑配置文件, 是其他应用内程序可识别Fcitx5中文输入
sudo vim .pam_environment
```
