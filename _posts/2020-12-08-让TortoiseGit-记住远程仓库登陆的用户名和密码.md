---
title: 让TortoiseGit-记住远程仓库登陆的用户名和密码
category: github
---

Windows 下 ctrl + r

运行框输入 %USERPROFILE%

新建文件 _netrc

内容如下

```
machine github.com
login 用户名
password 密码
```

