---
title: TortoiseGIt_SSH登陆
category: git
typora-root-url: ..
---

TortoiseGit基于putty的ssh实现密钥认证。



git bash生成密钥

```pretty
ssh-keygen -t ed25519 -C "your_email@example.com"
```

生成公钥和私钥

```
$ ls ~/.ssh/
id_ed25519  id_ed25519.pub 
```



1.3 将公钥加进git设置

![image-20201218221709446](/assets/img/image-20201218221709446.png)



2 客户端配置

因为TortoiseGit使用的密钥与git并不一样，它使用的是putty。要使用刚才生成的密钥，需要进行转换。

重新生成私钥

打开“puttygen.exe“

点击”load“，选择刚才的私钥文件id_ed25519，然后”save private key“保存成ppk文件。

2.3 git clone时指定私钥

![img](/assets/img/wKiom1hy8NHTetTVAACnXf4L7ZU449.png)

已克隆项目加载私钥

![image-20201218221915407](/assets/img/image-20201218221915407.png)