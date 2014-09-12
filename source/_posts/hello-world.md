title: hello world
---

终于下决心把博客由wordpress搬到github pages上了。。其实主要是懒得折腾。。这里把折腾[hexo](http://hexo.io/ "hexo")的过程记录一下，做个备用。

- #### 前提


1. 安装Git；
2. 安装Nodejs；


- #### 安装


1.  打开Git Bash，用npm安装hexo，步骤可以参考[hexo官方教程](http://hexo.io/)：
````
npm install hexo -g -verbose
````
*加verbose参数是为了查看详细信息，以免中途挂掉；*
*这里用的是Git Bash，不是cmd；*

2.  在本地创建目录：
````
hexo init content
````
*content为文件夹名，可以随意*
会在根目录创建一个content文件夹，这个文件夹就是hexo的工作目录：
![工作目录](http://upload-images.jianshu.io/upload_images/44230-4c07ec89090ea6f6.png)
![文件夹结构](http://upload-images.jianshu.io/upload_images/44230-ea6ab72972bff246.png)

3.  安装node缺失的依赖包
````
cd content
npm install -verbose
````
![安装依赖包](http://upload-images.jianshu.io/upload_images/44230-2a7d3f4ba53fcdb7.png)

4.  生成html静态页面，查看效果：
````
hexo generate
hexo server
````
![本地执行效果](http://upload-images.jianshu.io/upload_images/44230-f01b713d264754ab.png)
![文件夹结构](http://upload-images.jianshu.io/upload_images/44230-f6eb4485c95cb61f.png)
![生成的静态页面](http://upload-images.jianshu.io/upload_images/44230-1eb8b16b73831d83.png)


- #### 发布


1.  打开_config.yml，修改deploy节点：
````
deploy:
  type: github
  repo: https://github.com/imwower/imwower.github.io.git
````
*type和repo的冒号之后有一个空格*

2.  发布到github：
````
hexo deploy
````
*这里的发布使用的是https，需要手动输入github用户名和密码。ssh可能不太一样*

3. 创建另一个git repo，把当前文件夹内容提交，这样以后直接clone这个repo，就不需要再对hexo进行配置，同时可以随时修改博客内容了：
![另一个repo](http://upload-images.jianshu.io/upload_images/44230-aed8fffb03d3df60.png)

![上面的repo为工作目录，下面的repo为博客内容](http://upload-images.jianshu.io/upload_images/44230-6c60b2dbb17949a4.png)
