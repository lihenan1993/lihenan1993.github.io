---
title: logshtash-插件
category: ELK
typora-root-url: ../../
---

 # grok

1. 语法

   %{要匹配的值的正则:变量名}

   例如 %{info:level}

   将info字符串 匹配为等级

2. 每次只能处理文件中的一行
3. 有很多内置正则
4. ![image-20210514181551107](/C:/Users/86186/AppData/Roaming/Typora/typora-user-images/image-20210514181551107.png)