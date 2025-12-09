### 本文采用`Konsole` + `Zsh` + `Powerlevel10k`的搭配方式优化终端使用体验
- 参考自[archlinux 简明指南](https://arch.icekylin.online/guide/advanced/beauty-3.html)
## 1. 更换默认`Shell`为`zsh`
```bash
# 查看安装了哪些 Shell
chsh -l
# 如果没有安装zsh, 这里以arch为例执行安装命令
sudo pacman -Syu zsh
# 修改当前账户的默认 Shell
chsh -s /usr/bin/zsh 
```
## 2. 配置字体
- 使用`Powerlevel10k`进行配置时, 很多图标符号看不到，因为 powerlevel10k 中包含许多特殊图标符号，需要与之兼容的字体。
```zsh
# 搜索一下相关字体
yay -Ss nerd-font
# 程序员友好字体
sudo pacman -S ttf-jetbrains-mono-nerd
# Powerlevel10k官方推荐
sudo pacman -S ttf-meslo-nerd-font-powerlevel10k
```
- 安装完任意一个 Nerd Font 字体后，打开 Konsole 的 设置 > 编辑当前方案 > 外观，把 字体 改为刚刚安装的 Nerd Font 即可。
- 现在再打开 powerlevel10k 配置（p10k configure），就可以看到图标符号，正常配置了。