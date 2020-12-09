---
title: GitHub Page 使用
category: github
---



### GitHub Page

利用GitHub托管，搭建个人博客

### 环境准备

- 安装git

- 安装Ruby [Ruby安装教程](https://www.ruby-lang.org/zh_cn/documentation/installation/)

- 安装Jekyll

  ```shell
  gem install jekyll
  ```

### 搭建步骤

1. 创建git仓库

   在github以 username.github.io 作为仓库名，创建代码仓库

   例如 lihenan1993.github.io

2. 克隆仓库

   ```shell
   git clone https://github.com/lihenan1993/lihenan1993.github.io.git
   ```

3. 下载主题

   http://jekyllthemes.org/

   将主题的代码放入git克隆目录

4. ```shell
   cd lihenan1993
   # 需要翻墙
   bundle install
   # 启动Jekyll服务
   bundle exec jekyll serve
   ```

5. 本地访问 http://127.0.0.1:4000

6. 远程访问

   推送代码，然后访问

   https://lihenan1993.github.io

### 发布博客

进入 _post 目录

创建yyyy-MM-dd-filename.md

生成代码

```shell
bundle exec jekyll serve
```

推送代码