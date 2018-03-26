> 在使用HTML/CSS/JS设计软件界面时，如果每次都从远程服务器加载资源那显然是不现实的：如果界面是从远程服务器获取，遇到网速不佳时，那么将给用户带来非常糟糕的体验。为了避免这种尴尬的局面，NanUI可以将呈现界面的HTML/CSS/JS等文件作为嵌入资源编译到项目中，这样操纵既可以加速软件界面的加载速度，也可以避免界面遭到恶意修改。
>
> 下面的文章将详细介绍如何将HTML/CSS/JS等文件打包到项目中。

## 第一步 新建NanUI项目

按照之前的教程，新建一个Windows窗体应用程序，并在Program中完成CEF的相关设置操作。

## 第二步 在项目中添加网页资源

```
<!DOCTYPE html>
<html lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
	<meta charset="utf-8" />
	<title>HELLO</title>
</head>
<body>
	<h1>Hello NanUI!</h1>
</body>
</html>
```

若要将index.html文件作为嵌入资源编译到项目中，您需要在Visual Studio的\*\*资源管理器\*\*中选中index.html文件，然后在该文件的\*\*属性窗口\*\*的\*\*生成操作\*\*栏目中选中该文件为\*\*嵌入的资源\*\*。如图所示：

![](http://img.blog.csdn.net/20171226231130613?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGlueHVhbmNoZW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这样，在项目编译时index.html文件将会作为内嵌的资源编译到输出的可执行文件中。

# 第三步 注册嵌入资源的程序集



