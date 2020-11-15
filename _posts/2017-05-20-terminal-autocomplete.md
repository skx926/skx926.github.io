---
title: 让Mac终端自动补全忽略大小写
date: 2017-05-20 11:55:33
tags: [Mac, 终端]
categories: [Mac]
---

Mac自带的 `Terminal.app` 已经有自动补全功能，但是需要区分大小写。只需要在用户目录下修改 `.inputrc` 文件的配置就可以让它忽略大小写。具体操作如下：

```bash
$ vi ~/.inputrc
```

然后在里面添加下面的内容：

```bash
set completion-ignore-case on
set show-all-if-ambiguous on
TAB: menu-complete
```

保存后重新打开终端就可以使用 `Tab` 键愉快的使用了。
