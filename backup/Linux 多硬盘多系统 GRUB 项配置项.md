# GRUB启动项个人配置记录

本文基于[GRUB配置文档](https://wiki.archlinux.org.cn/title/GRUB)、[GRUB/技巧与提示](https://wiki.archlinux.org.cn/title/GRUB/Tips_and_tricks). 提取出个人需要的部分做配置记录

## 1. 系统硬盘情况
- 硬盘 1：500GB，Linux 安装( EndeavourOS )
- 硬盘 2：256GB，Windows 安装( Windows 10 )

> [!TIP]
> 说明：在多硬盘多系统环境下，GRUB 推荐安装在 Linux 系统所在硬盘的 EFI 分区或者 MBR 上，以便管理所有系统启动项。

## 2. GRUB 配置文件
### 2.1 主要文件位置
- `/etc/default/grub`：GRUB 主配置文件（设置默认启动、等待时间、内核参数等）
- `/etc/grub.d/`：各个启动项脚本（Linux 内核、其他操作系统）
- `/boot/grub/grub.cfg`：最终生成的 GRUB 配置文件，不建议手动修改，由 `update-grub` 自动生成
- `/etc/grub.d/40_custom`：用户手动添加自定义启动项的地方


### 2.3 编辑配置文件
#### 基本配置
在 /etc/default/grub 中设置默认启动选项, 等待时间：
```vim
GRUB_DEFAULT=saved      # 上一次启动的系统
GRUB_SAVEDEFAULT=true   # 自动保存最后一次启动项
GRUB_TIMEOUT=3         # 等待 3 秒
GRUB_TIMEOUT_STYLE=menu # 显示菜单 - 一般情况EndeavourOS为menu, 无需修改
```
#### 可选配置
设置主题
```vim

```
> [!NOTE]
> 若配置完`记住上一次启动的系统`且 `更新 GRUB` 后 依旧不生效, 尝试编辑`40_custom`, 文件末尾添加
> ```bash
> menuentry <你的系统在GRUB显示的名字, 例如Windows Boot Manager (Custom)> --class windows --class os {
>     savedefault
>     insmod part_gpt
>     insmod fat
>     search --no-floppy --fs-uuid --set=root <你的Windows分区UUID>
>     chainloader /EFI/Microsoft/Boot/bootmgfw.efi
> }
> ```
> 可以通过以下命令查询Windows分区UUID, 例如
> ```bash
> lsblk -f
>```
> 输出一些内容, 其中nvme1n1p1是我安装Windows的硬盘, 对应的分区UUID为BC83-5841
> ```bash
> NAME        FSTYPE FSVER LABEL       UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
> nvme0n1                                                                                  
> ├─nvme0n1p1 vfat   FAT32             BFE4-72E8                                 2G     0% /boot/efi
> └─nvme0n1p2 ext4   1.0   endeavouros 881aa179-db4f-4240-b151-ac67f02dbb47  413.7G     4% /
> nvme1n1                                                                                  
> ├─nvme1n1p1 vfat   FAT32             BC83-5841                                           
> ├─nvme1n1p2                                                                              
> ├─nvme1n1p3 ntfs                     C05884BF5884B5A6                                    
> └─nvme1n1p4 ntfs                     F29CA6EF9CA6AD93                                    


### 2.3 更新 GRUB
```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
