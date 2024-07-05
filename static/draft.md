# 掌控Vim：光标操作与快捷键的终极指南

## 写在前面
最近在学习python, 环境用的是云端linux服务器, 跟着oeasy老师用vim编辑器, 碍于之前用git合并时,命令行总是弹出的提示输入merge信息, 默认也是vim的文本编辑器, 导致只会`:q`的我每次都不写merge info, 因为一旦进入了vim的insert模式, 我就不能通过 `ctrl` `shift` `home/end` `↑/↓/←/→` 这些按键的组合键方式移动光标了, 一旦写错, 编辑起来十分痛苦.

每次看到有大佬用vim写代码都有一种敬畏感, 之前读过一篇[采访Snipaste开发者](https://sspai.com/post/35097)的文章, 其中有句话我很认同:

```txt
如果想要学会一个高级工具，那学习成本自然也是要的，世界上并不存在不需要学习成本但却很强大的工具。
强大的工具通常还有一个共性，就是一旦你掌握了它，它带给你的回报远远超过你当时付出的时间和精力成本。
```

当我知道了vim编辑熟练后, 甚至手腕都不需要怎么移动, 就可以完成几乎所有常见操作的时候.  更加好奇其操作方式, 故记录于此.





```
> [!NOTE]
> Useful information that users should know, even when skimming content.

> [!TIP]
> Helpful advice for doing things better or more easily.

> [!IMPORTANT]
> Key information users need to know to achieve their goal.

> [!WARNING]
> Urgent info that needs immediate user attention to avoid problems.

> [!CAUTION]
> Advises about risks or negative outcomes of certain actions.
```

![image-20240705151829330](/Users/piaoyangguohai/Library/Application Support/typora-user-images/image-20240705151829330.png)
