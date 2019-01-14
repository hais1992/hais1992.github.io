---
title: 新电脑发布文章的方法
date: 2017-09-15 10:52:38
tags: [Hexo,新电脑,发布,文章]
categories: ["其它"]
---

1、安装 Node.js 和 Git
``` bash
下载地址：http://nodejs.org/
下载地址：http://git-scm.com/
```

2、安装Hexo
``` bash
npm install -g hexo-cli
```

3、用git拉下hexo分支代码
``` bash
$ git clone -b hexo git@github.com:hais1992/hais1992.github.io.git
```

4、安装环境
``` bash
$ cd hais1992.github.io.git
$ npm install hexo
$ hexo init
$ npm install
$ npm install hexo-deployer-git
```

5、然后就可以编辑文章发布了
``` bash
$ hexo generate -d
```

6、发布完成后就可以继续提交代码备份了
``` bash
$ git add *
$ git commit -m 提交代码
$ git push
```


参考地址：
	https://hexo.io/zh-cn/docs/setup.html
	http://crazymilk.github.io/2015/12/28/GitHub-Pages-Hexo%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2/#more
	https://github.com/Ben02/hexo-theme-Anatole/wiki/Installation
	