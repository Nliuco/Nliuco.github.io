# GRUB启动项个人配置记录

本文基于[GRUB官方文档](https://wiki.archlinux.org.cn/title/GRUB)、[GRUB/技巧与提示](https://wiki.archlinux.org.cn/title/GRUB/Tips_and_tricks). 提取出个人需要的部分做配置记录

## 1. 系统硬盘情况
- 硬盘 1：500GB，Linux 安装
- 硬盘 2：256GB，Windows 安装

> [!TIP]
> 说明：在多硬盘多系统环境下，GRUB 推荐安装在 Linux 系统所在硬盘的 EFI 分区或者 MBR 上，以便管理所有系统启动项。

## 2. GRUB 配置文件
### 2.1 主要文件位置
- `/etc/default/grub`：GRUB 主配置文件（设置默认启动、等待时间、内核参数等）
- `/etc/grub.d/`：各个启动项脚本（Linux 内核、其他操作系统）
- `/boot/grub/grub.cfg`：最终生成的 GRUB 配置文件，不建议手动修改，由 `update-grub` 自动生成

### 2.2 更新 GRUB
```bash
sudo update-grub      # Ubuntu/Debian 系列
sudo grub2-mkconfig -o /boot/grub2/grub.cfg  # CentOS/Fedora