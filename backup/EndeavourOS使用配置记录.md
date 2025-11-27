# EndeavourOS使用过程中遇到的问题
## 服务不可用
### 1. 蓝牙无法正常打开, 需要安装额外包
- 先查看蓝牙服务是否正常启用, 尝试使用`systemctl`重新启用
```bash
systemctl status bluetooth.service # 状态
sudo pacman -S bluez bluez-utils
sudo systemctl enable --now bluetooth.service
systemctl status bluetooth
```