<p align="center">
  <img src="https://assets.pianone.dpdns.org/file/pic/blog/ptzZeMbB.svg" alt="obsidian-logo-text-white-purple.svg" width="50%">
</p>

> [!NOTE]
> 对于安卓端的操作，建议提前配置好 PC 端（参考：[Obsidian Git 云同步配置（PC 端）](https://blog.pianone.dpdns.org/post/Obsidian%20Git%20-yun-tong-bu-pei-zhi-%28PC%20-duan-%29.html)），这样会省去仓库初始化、Obsidian 插件下载与配置等一系列操作，便于安卓端快速集成使用。下文操作都基于 PC 端已配置好的前提，记录了如何在安卓端进行同步工作的教程。

## 一、安装与拉取仓库

### 1. 下载

可以去 GitHub 仓库主页或者在 Google Play 应用市场中搜索 `Git Sync`，安装到手机上。

### 2. 快速设置

打开下载好的 `Git Sync` 应用：
- 跳过新手教程引导步骤，给予必要的文件访问权限即可
- 快进到 `Set up` 操作

### 3. 身份验证

默认可以直接使用 `GitHub` 的 `OAuth 提供商`进行身份验证。不过由于 PC 端是在 `Gitee` 上进行的仓库创建和初始化，而 `Git Sync` 并没有直接提供 `Gitee` 的 `OAuth 提供商`，所以我们选择 `GIT 协议` 进行认证，本文选择使用 `GIT 协议` 下的 `HTTPS` 协议进行身份验证。

选择后，提示需要输入 `用户` 和 `令牌`：

- **`用户`**：如果 Gitee 账号在 `设置` > `个人资料` 里面曾经设置过用户名称，一定要区分这里输入的用户字段是**手机号**而非用户名称，否则无论如何也不能认证成功。
- **`令牌`**：需要到 [设置私人令牌页面](https://gitee.com/profile/personal_access_tokens) 生成一个新令牌，保证令牌拥有合法访问仓库并拉取/推送代码的权限即可；当然，也可以另外设置一个有效期以增强安全性。

输入完成后会出现登录按钮，点击登录按钮会跳转到克隆远程仓库页面。

> [!WARNING]
> `Git Sync` 并不会在此时直接验证输入的用户和令牌是否有效，而是在后面的仓库 clone 时才会提示错误信息，一定要确认好用户名和令牌的正确性！

### 4. Clone 仓库

1. 将自己 Gitee 仓库的 clone 地址复制到 `Git Sync` 的输入栏，点击 `克隆` 按钮。
2. 此时会提供 `直接克隆` 或 `嵌套克隆` 两种方式（具体请查看其弹窗描述），选择其一并指定一个本地 clone 位置即可。

## 二、手机端 Obsidian 禁用 Git 插件

当 clone 完毕仓库之后，`Git Sync` 会检测到我们使用了 Obsidian Git 插件（即之前在 PC 端下载的 Obsidian Git 同步插件）。由于在手机上我们使用 `Git Sync`，二者同时使用可能会有冲突，详见下方 Git Sync 官方文档链接。

**具体操作**：在手机上先打开 Obsidian 的 `Git` 设置，然后启用 **“在此设备上禁用”** 选项。

> [!IMPORTANT]
> 相关说明可参考 Git Sync 官方文档：[文档地址](https://gitsync.viscouspotenti.al/wiki/troubleshooting#concurrent-repository-access)

## 三、启用自动同步

`Git Sync` 提供了很多种自动同步的方式，这里我推荐两种：

1. **打开或关闭应用进行同步**：可以选择监听的应用 `Obsidian`，等我们手动打开或者关闭 Obsidian 的时候，`Git Sync` 就会自动帮我们拉取最新的笔记内容并提交本地的变更信息到远程。
2. **计划同步**：可以设置定时多久在后台自动执行一次同步，这里我设置为 `12 小时` 自动同步一次，避免没有打开软件的情况下太长时间仓库不更新。

> [!NOTE]
> 这里需要给 `Git Sync` 启用手机系统的**无障碍功能**，才能保证自动同步功能的正常使用。

到此为止，App 端就可以正常获取最新的笔记内容，在 App 端的变更也可以自动提交到远程分支了~
