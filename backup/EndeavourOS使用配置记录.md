# EndeavourOS使用过程中遇到的问题
## 服务不可用
### 1. 蓝牙无法正常打开
- 先查看蓝牙服务是否正常启用, 尝试使用`systemctl`重新启用, 亦或是需要安装额外包
```bash
systemctl status bluetooth.service # 状态
sudo pacman -S bluez bluez-utils
sudo systemctl enable --now bluetooth.service
systemctl status bluetooth
```
### 2. 输入法不自带中文输入
- 使用Fcitx5输入框架, 安装Rime, 根据个人使用习惯,  配置小鹤双拼输入
> [!NOTE]
> 采用[凇鹤拼音](https://github.com/kchen0x/rime-crane) —— Rime 简体中文输入法方案，整合了雾凇拼音和小鹤双拼/音形方案的拼音输入法。
```bash
sudo pacman -S fcitx5-im fcitx5-chinese-addons
paru -S fcitx5-skin-ori-git # 输入法主题-可选
```
