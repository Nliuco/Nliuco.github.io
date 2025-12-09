# 本文采用`Konsole` + `Zsh` + `Powerlevel10k`的搭配方式优化终端使用体验
## 1. 更换默认`Shell`为`zsh`
```bash
# 查看安装了哪些 Shell
chsh -l
# 如果没有安装zsh, 这里以arch为例执行安装命令
sudo pacman -Syu zsh
# 修改当前账户的默认 Shell
chsh -s /usr/bin/zsh 
```
## 2. 