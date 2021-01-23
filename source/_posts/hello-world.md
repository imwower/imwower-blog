---
title: 使用GitHub Pages托管博客
date: 2020-05-08 23:39:21
tag: [hello-world, hexo, GitHub, highlight]

---

[GitHub Pages](https://pages.github.com/ "GitHub Pages")本身可以托管静态页面，也绑定自定义域名，同时支持SSL证书；这里使用[Hexo](http://hexo.io/ "hexo")生成静态页面，然后托管在GitHub上。以下是折腾过程，做个备用。

#### 前提

1. 本地安装Git；
2. 本地安装Node.js；

#### 安装

1. 命令行执行npm安装Hexo，步骤可以参考：[Hexo官方教程](http://hexo.io/)；
	加`verbose`参数是为了查看详细信息，以免中途挂掉：
``` bash
npm install hexo-cli -g -verbose
```

2. 在本地创建博客的根目录，会创建一个`blog`文件夹，这个文件夹就是Hexo的工作目录：
``` bash
hexo init blog
```

3. 
``` bash
cd blog
ls
_config.yml  db.json        package.json       public/     source/
CNAME        node_modules/  package-lock.json  scaffolds/  themes/
```

4. 安装`Hexo`缺失的依赖包:
``` bash
cd blog
npm install -verbose
```

5. 生成html静态页面，本地查看效果：
````
hexo generate
hexo server
````
	*或者使用简写命令：`hexo g & hexo s`*

#### 发布
1. 这里发布到GitHub Pages，还需要安装[hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git)包：
```
npm install hexo-deployer-git --save
```

2. 打开`_config.yml`，修改`deploy`节点：
```
deploy:
  type: git
  repo: https://github.com/imwower/imwower.github.io.git
```

3. 发布到GitHub：
```
##  这里其实就是把`public`文件夹里的所有静态页面提交到GitHub上了而已
hexo deploy
```

4. 创建另一个Git repo，把当前文件夹(`blog`)内容提交，这样以后直接修改`blog`这个repo，就不需要再对Hexo进行配置，包括主题、包依赖等，同时可以随时修改博客内容了：
```
工作目录(blog）: [https://github.com/imwower/imwower-blog.git](https://github.com/imwower/imwower-blog.git)
博客内容: [https://github.com/imwower/imwower.github.io.git](https://github.com/imwower/imwower.github.io.git)
```

#### 其他
1. 修改`blog`目录下的`_config.yml`，使用`iceman`主题：
```
theme: iceman
```

2. 关闭自带的语法高亮插件，使用`highlightjs`：
``` yml
##  将默认的`highlight`修改为`false`
highlight:
	enable: false
	line_number: true
	auto_detect: false
 	tab_replace: ''
	wrap: true
	hljs: false
```

3. 添加新的配置项：
``` yml
highlightjs: true
```

4. 进入主题所在的文件夹，找到`/blog/themes/iceman/layout/_partial/footer.ejs`模板，添加`highlight.js`的引用：
``` html
	<div id="footer">
	...省略中间代码...
	</div>

	<% if (config.highlightjs) { %>
	<!-- Highlight.js -->
	<link rel="stylesheet"
	      href="https://cdn.bootcdn.net/ajax/libs/highlight.js/10.0.1/styles/atom-one-dark.min.css">
	<script src="https://cdn.bootcdn.net/ajax/libs/highlight.js/10.0.1/highlight.min.js">
	</script>
	<script>
	    hljs.initHighlightingOnLoad();
	</script>
	<% } %>
```

	*参考：[Hexo博客添加highlight-js代码高亮](https://zihengcat.github.io/2018/03/05/Hexo%E5%8D%9A%E5%AE%A2%E6%B7%BB%E5%8A%A0highlight-js%E4%BB%A3%E7%A0%81%E9%AB%98%E4%BA%AE/)*


5. 博客插入图片：
这里是直接在`/blog/source/images/`文件夹下存放图片文件，在文章里插入图片时，使用相对路径：
```
markdown代码：`![image](/images/2.png)`
```
	示例：![image](/images/2.png)