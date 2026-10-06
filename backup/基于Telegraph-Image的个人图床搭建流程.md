# 个人图床搭建流程

> [!WARNING]
> 搭建前提醒
> 必需: GitHub账户; Cloudflare账户; Telegram账号
> 可选: 域名

## 一、部署流程

### 1. 在Github中Fork 仓库

打开 [Telegraph-Image](https://github.com/cf-pages/Telegraph-Image) 项目主页，点击 **Fork** 将仓库复制到自己的 GitHub 账号下，后续部署都基于这份 fork 进行。
可以顺手点个Star :stuck_out_tongue_winking_eye: , 主要是后续可以方便找README看文档~

### 2. Cloudflare 部署

1. 打开 [Cloudflare](https://dash.cloudflare.com/) 主页，进入 **Workers 和 Pages**
2. 进入 **Pages**，先连接到 Git
3. 选择刚刚 fork 的仓库，点击部署，稍等片刻即可部署成功

### 3. 获取 Telegram 密钥

1. 打开 Telegram，搜索 `@BotFather`，或者用中文的 `@机器人之父`
2. 发送 `/newbot`，创建新的机器人，会自动获取 **TG_Bot_Token**
3. 将上述值先保存好，稍后配置到 Cloudflare 中
4. 创建一个新的频道，并设置机器人为频道管理员
5. 搜索 `@VersaToolsBot` 来获取频道 ID（**TG_Chat_ID**）
    - 转发新创建频道的一条消息即可（`@GetTheirIDBot` 亦可）

### 4. 回到 Cloudflare 完成配置

1. 首先是去设置页面，配置 `TG_Bot_Token` 和 `TG_Chat_ID`
2. 现在可以先去重新部署一下，测试图片上传功能是否已经正常可以使用了
3. 如果正常，为了使用管理后台的功能，就进行接下来的工作，创建 KV 命名空间
    - 鉴于 Cloudflare 偶尔改版，菜单位置可能变动，更推荐搜索 KV 打开页面
    - 创建一个 KV 命名空间，比如说名字叫 `telegraph-image`
4. 回到项目设置页，在【绑定】下添加【KV 命名空间】，名称为 `img_url`，值选择刚刚创建的 KV

到目前为止，图床和管理后台就已经是可用的了。

### 5. 配置自定义域名（可选）

前提是有一个托管在 Cloudflare 上的域名。部署成功后，Pages 的部署页面会直接出现「添加自定义子域」的下一步提示：

1. 点击进入「添加自定义子域」
2. 因为域名和项目都在 Cloudflare 上，CNAME 记录的名称和值会自动填好，小橙云（代理）也会自动开启，点两下确认即可完成

配置完成后，DNS 记录里会多出这样一条(备注是我自己后加的)：

| 名称                  | 类型    | 内容                            | 代理状态     | 备注                           |
| ------------------- | ----- | ----------------------------- | -------- | ---------------------------- |
| assert.your.domain.com | CNAME | your.cf.example.pages.dev | 已代理（小橙云） | 基于Telegraph-Image的个人图床的自定义域名 |

> [!NOTE]
> 对图床来说，小橙云（代理）建议保持开启。
> 如果域名没有托管在 Cloudflare，则需要自己手动添加上面这条 CNAME 记录。

子域按喜好起名即可，这里用的是 `assert`, 最终图床地址为 `https://assert.your.domain.com`。

## 二、可选配置

> 查看 Telegraph-Image 项目的 [README.md](https://github.com/cf-pages/Telegraph-Image/blob/main/README-zh.md)
> 有很多可选参数，我们可以灵活配置一些额外的功能

### 1. 后台管理页面的账户密码配置

| 环境变量     | 示例值           | 说明                                                       |
| ---          | ---              | ---                                                        |
| `BASIC_USER` | `admin`          | 后台管理页面（/admin）的登录用户名。不设置则后台无需登录。 |
| `BASIC_PASS` | `admin-password` | 后台管理页面的登录密码，需要和 `BASIC_USER` 同时设置。     |

### 2. 上传入口的账户密码配置

| 环境变量            | 示例值            | 说明                                                              |
| ---                 | ---               | ---                                                               |
| `UPLOAD_BASIC_USER` | `uploader`        | 上传入口的 Basic Auth 用户名。不设置则保持公开上传。              |
| `UPLOAD_BASIC_PASS` | `strong-password` | 上传入口的 Basic Auth 密码，需要和 `UPLOAD_BASIC_USER` 同时设置。 |

### 3. 开启短链接

| 环境变量            | 示例值 | 说明                                                                            |
| ---                 | ---    | ---                                                                             |
| `ENABLE_SHORT_URLS` | `true` | 开启后（需绑定 KV）上传将返回形如 `/file/AbC123` 的短链接，原有长链接依然有效。 |

### 4. 开启图片审查服务

> [!TIP]
> 该项不在全局设置里, 在 `设置`->`函数`->`Workers AI 绑定`, 添加一个变量名称为 `AI` 的绑定

| 环境变量              | 示例值          | 说明                                                                                                                                                                                                                                                                                                                               |
| ---                   | ---             | ---                                                                                                                                                                                                                                                                                                                                |
| `MODERATION_PROVIDER` | `cloudflare-ai` | 图片审查服务：`cloudflare-ai`（Workers AI，推荐）、`moderatecontent`（旧版）或 `none`。不设置时自动检测：有 `ModerateContentApiKey` 用 moderatecontent，有 `AI` 绑定用 Workers AI。详见[开启图片审查](https://github.com/cf-pages/Telegraph-Image/blob/main/README-zh.md#%E5%BC%80%E5%90%AF%E5%9B%BE%E7%89%87%E5%AE%A1%E6%9F%A5)。 |

### 5. 配置站点名称和首页浏览器标签页标题

| 环境变量     | 示例值             | 说明                                                          |
| ---          | ---                | ---                                                           |
| `SITE_NAME`  | `Pianone Images`        | 首页顶部显示的站点名称（通过 `GET /api/config` 下发给前端）。 |
| `SITE_TITLE` | `Pianone Images \| Home` | 首页的浏览器标签页标题。                                      |

### 6. 配置首页背景图

| 环境变量          | 示例值               | 说明             |
| ---               | ---                  | ---              |
| `SITE_BACKGROUND` | `https://.../bg.jpg` | 首页背景图 URL。 |

### 7. 隐藏首页上的后台入口链接

| 环境变量           | 示例值 | 说明                                                  |
| ---                | ---    | ---                                                   |
| `HIDE_ADMIN_ENTRY` | `true` | 隐藏首页上的后台入口链接（/admin 页面本身仍可访问）。 |

> [!NOTE] 
> 此外, 还有很多其他设置, 例如白名单/防盗链等功能
> 不过我并没有全部使用过, 这里只列出一些自己用过的功能.

## 三、踩坑记录

**Q:** 在 [6. 配置首页背景图](#6.-配置首页背景图) 的时候, 遇到了配置不生效的问题?

**A:** 当前版本的 `index.html` 中存在 `loadWallpapers` 逻辑，即使配置了 `SITE_BACKGROUND`，首页仍然会加载 Bing 壁纸。(编写本文时, 最新分支是 ef69018df1967708ec026a671e3976f5524ff490, Commits on Jul 25, 2026)

我的解决方式是：

1. 为了避免影响 `main` 分支后续与上游仓库同步，单独创建 `deploy` 分支。
2. 修改 `index.html`，注释掉 `loadWallpapers` 相关代码，只保留 `api/config` 的背景图配置。
3. 将修改后的代码保留在 `deploy` 分支。
4. 在 Cloudflare Pages 中将生产部署分支修改为 `deploy`。

最终：

```text
main    → 保持与上游仓库同步
deploy  → 保存个人定制修改，并用于 Cloudflare Pages 部署
```

这样既可以使用自己的首页背景图，又不会直接修改 `main` 分支，后续同步上游仓库时也更加方便。

现在你可以好好享受自己的图床了~

## 参考鸣谢

- [Telegraph-Image](https://github.com/cf-pages/Telegraph-Image)
- [程序员哈利的博客](https://hali.life/?p=59)
