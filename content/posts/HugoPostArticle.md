---
title: Hugo发布一篇新文章
date: 2019-01-14T10:26:34+08:00
tags: [Hexo,新电脑,发布,文章]
categories: ["其它"]
---

本地调试
``` bash
hugo server --buildDrafts
```

生成网站
``` bash
hugo --theme=LeaveIt --baseUrl="http://hais.pw/"
```

部署、上传
``` bash
git add -A
git commit -m "修改说明"
git push
```
