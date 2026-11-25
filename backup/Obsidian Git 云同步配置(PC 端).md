<p align="center">
  <img src="https://assets.pianone.dpdns.org/file/pic/blog/ptzZeMbB.svg" alt="obsidian-logo-text-white-purple.svg" width="50%">
</p>

> [!NOTE]
> 这里推荐国内用户使用 [Gitee](https://gitee.com/) 作为笔记(代码)托管平台

## 一、准备工作
### 1. 安装git
略, 见: [安装Git](https://git-scm.com/install/), 本文作者使用scoop管理Git

### 2. 注册Gitee
打开 [Gitee](https://gitee.com/)官网进行注册, [配置SSH Key](https://help.gitee.com/enterprise/code-manage/%E6%9D%83%E9%99%90%E4%B8%8E%E8%AE%BE%E7%BD%AE/%E9%83%A8%E7%BD%B2%E5%85%AC%E9%92%A5%E7%AE%A1%E7%90%86/%E7%94%9F%E6%88%90%E6%88%96%E6%B7%BB%E5%8A%A0SSH%E5%85%AC%E9%92%A5)

### 3. 创建 Gitee 仓库
进入：[Gitee新建仓库](https://gitee.com/projects/new)
创建：
```text
仓库名：obsidian-notes
类型：私有仓库（推荐）
```
> 不要初始化 README。

创建完成后会得到：
```bash
https://gitee.com/你的用户名/obsidian-notes.git
```
或者 SSH：
```bash
git@gitee.com:你的用户名/obsidian-notes.git
```
推荐 SSH。

### 4. 生成 SSH Key
Mac / Linux / Windows PowerShell：
```bash
ssh-keygen -t ed25519 -C "你的邮箱"
```
一路回车。
生成后：
```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```
### 5. 查看公钥
```bash
cat ~/.ssh/id_ed25519.pub
```
复制全部内容。

### 6. 添加到 Gitee
打开：[Gitee SSH Keys设置](https://gitee.com/profile/sshkeys), 添加公钥。

### 7. 测试 SSH
```bash
ssh -T git@gitee.com
```
看到：
```text
Hi xxx! You've successfully authenticated...
```
即可。

---

## 二、构建仓库
### 1. 本地初始化 Obsidian Git 仓库
假设你的 Obsidian 仓库目录：
```text
~/Documents/ObsidianVault
```
进入目录：
```bash
cd ~/Documents/ObsidianVault
```
初始化 Git：
```bash
git init
```
关联远程仓库：
```bash
git remote add origin git@gitee.com:你的用户名/obsidian-notes.git
```
### 2. 添加 .gitignore
创建：
```bash
touch .gitignore
```
内容：
```gitignore
.obsidian/workspace.json
.obsidian/cache
.trash
.DS_Store
```
> 说明：
> - workspace.json 不同步布局
> - cache 不需要同步
> - 避免 macOS 垃圾文件

### 3. 首次提交
```bash
git add .
git commit -m "init obsidian vault"
git branch -M main
git push -u origin main
```
现在你的笔记已经到 Gitee 了。

---

## 三、插件设定
### 1. 安装 Obsidian Git 插件
打开 Obsidian：
```text
设置
→ 第三方插件
→ 关闭安全模式
→ 浏览插件
```
搜索：
```text
Git
```
安装并启用。

### 2.配置 Obsidian Git 插件
进入：
```text
设置
→ Git
```
推荐配置：
#### 1)自动 Pull
开启：
```text
Pull updates on startup
```
作用：打开 Obsidian 自动拉取最新笔记。

#### 2)自动 Push
开启：
```text
Vault backup interval
```
推荐：
```text
1 分钟
```
表示：
```text
自动 commit + push
```
#### 3)自动同步
建议开启：(默认应该是已开启状态)
```text
Auto pull before push
```
避免冲突。

---
## 四、多端同步
另一台电脑：
### 1. Clone 仓库
```bash
git clone git@gitee.com:你的用户名/obsidian-notes.git
```
### 2. 用 Obsidian 打开目录
### 3. 信任 Obsidian Git 插件
之后重复前面的插件配置。

---
## 五、常见问题

### 配置Commit 用户名/邮箱
```bash
	git config --global user.name '用户名'
	git config --global user.email 'Gitee账号注册用的邮箱'
```

> [!TIP]
> 如果不想改全局的用户名/邮箱, 删掉--global参数, 去对应Obsidian 仓库目录下执行命令即可
