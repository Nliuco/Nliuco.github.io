# 本文采用`Konsole` + `Zsh` + `Powerlevel10k`的搭配方式优化终端使用体验
## 1. 更换默认`Shell`为`zsh`
```bash
chsh -l # 查看安装了哪些 Shell
# 如果没有安装zsh, 这里以arch为例执行安装命令
sudo pacman -Syu zsh
chsh -s /usr/bin/zsh # 修改当前账户的默认 Shell
```
## 2. 