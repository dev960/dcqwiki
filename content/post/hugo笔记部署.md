---
title: "Hugo+Github个人笔记部署简易教程"
date: 2023-09-10T01:37:56+08:00
lastmod: 2023-09-10T01:37:56+08:00
draft: false
tags: ["hugo", "笔记", "主题"]
categories: ["笔记"]
author: "dcq"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'

---

#### hugo笔记部署

1. sudo apt install hugo -y

2. hugo new site dcqwiki

3. cd dcqwiki

4. git git submodule add https://github.com/olOwOlo/hugo-theme-even/tree/v4.0.0https://github.com/olOwOlo/hugo-theme-even themes/even

5. hugo server

6. 本地访问localhost:1313

7. 编译：hugo

8. github创建仓库

9. 名字和github名字一样

10. 勾选publi和Readme

11. cd public

12. git init -b main

13. git remote add origin git@github.com:dev960/dev960.github.io.git

14. git pull --rebase origin main

15. git add .

16. git commit -m "first commit"

17. git push origin main

18. hugo server -t even --buildDrafts 

> 指定模板渲染文章
