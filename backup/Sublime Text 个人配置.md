## 1. 插件

### 1.1 安装插件

1. 打开命令面板：`Ctrl+Shift+P`
2. 输入并选择 `Package Control: Install Package`，回车
3. 在弹出输入框里输入插件名（如 `MarkdownPreview`）
4. 从结果中选中对应插件，回车即开始安装
5. 装完即自动启用，多数插件无需重启

> 如果还没装 Package Control：先到官网复制安装命令，打开 `View > Show Console`，把命令黏贴进去回车即可。

### 1.2 插件一览

| 插件                     | 说明                           |
| ---------------------- | ---------------------------- |
| Package Control        | 插件包管理器，装/卸插件的入口，必备           |
| A File Icon            | 侧边栏按文件类型显示彩色图标               |
| AutoFileName           | 输入文件名/路径时自动补全（如 `src="..."`） |
| AutoHotkey             | AHK 脚本语法高亮                   |
| BracketHighlighter     | 高亮当前括号/引号/标签的配对位置            |
| ChineseLocalizations   | 界面菜单汉化（中文）                   |
| ConvertToUTF8          | 自动转 UTF-8，处理 GBK 等旧编码不乱码     |
| LiveReload             | 保存后自动刷新浏览器预览                 |
| Markdown               | Markdown 语法高亮基础支持            |
| MarkdownEditing        | 增强 Markdown 编辑（配色、输入辅助）      |
| MarkdownPreview        | 浏览器/内置渲染预览（`alt+m` 就是它）      |
| Material Theme         | 材质主题套装（Palenight 配色就是它）      |
| PowerShell             | PowerShell 语法高亮与片段           |
| SideBarEnhancements    | 侧边栏右键增强（新建/改名/移动/复制路径）       |
| SublimeAStyleFormatter | 格式化 C/C++/Java 等源码           |
| Terminus               | 内置终端                         |
| ToDone                 | 文档里做 TODO 任务清单               |

### 1.3 插件配置
MarkdownPreview：
```sublime-settings
{
	"enable_autoreload": true
}
```
作用：保存 Markdown 后浏览器预览自动刷新。

### 1.4 配置快捷键
```sublime-keymap
[
    // Markdown Preview
	{   
        "keys": ["alt+m"], 
        "command": "markdown_preview", 
        "args": {
            "target": "browser", 
            "parser":"markdown"
        }
	},
    // 打开终端
    {
        "keys": ["alt+f12"],
        "command": "toggle_terminus_panel",
        "args" : {
            "cmd": "pwsh.exe"
        }
    },
    // 格式化
    {
        "keys": ["ctrl+alt+l"],
        "command": "reindent",
        "args" : {
            "single_line": false,
        }
    },
]
```

## 2. 设置选项
```sublime-settings
{

	// ---- 禁用的内置功能 ----
	// 忽略 Vintage 包，禁用 Vim 模式
	"ignored_packages":
	[
		"Vintage",
	],
	// 不索引文件
	"index_files": false,

	// ---- 界面外观 ----
	// 字体
	"font_face": "JetBrains Mono",
	// 字体大小
	"font_size": 16,
	// 设置光标闪动方式 smooth, phase, blink, solid
	"caret_style": "smooth",
	// 滚动条自动隐藏显示
	"overlay_scroll_bars": "enabled",
	// 突出显示当前光标所在的行
	"highlight_line": false,
	// 宽度指导线
    // "rulers": [80, [100, "dotted", 1]],
    // 配色
	"color_scheme": "Mariana.sublime-color-scheme",
	// 主题
	"theme": "Material-Theme-Palenight.sublime-theme",
	
	// ---- 编辑行为 ----
	// Tab 宽度为 4 个空格
	"tab_size": 4,
	// 自动缩进大小为 4
	"indent_size": 4,
	// 将Tab转换为空格
	"translate_tabs_to_spaces": true,
	// 自动换行
	"word_wrap": false,
	// 自动补全
	"auto_complete": true,
	// 自动匹配括号、引号
	"auto_match_enabled": true,
	
	// ---- 文件与存档 ----
	// 失去焦点自动保存
	"save_on_focus_lost": true,
	// 退出时保留未保存内容，下次启动恢复
	"hot_exit": true,
	// 窗口右下角显示打开文件的编码
	"show_encoding": true,
}
```

