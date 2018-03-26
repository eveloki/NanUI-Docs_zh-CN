在使用HTML/CSS/JS设计软件界面时，如果每次都从远程服务器加载资源那显然是不现实的：如果界面是从远程服务器获取，遇到网速不佳时，那么将给用户带来非常糟糕的体验。为了避免这种尴尬的局面，NanUI可以将呈现界面的HTML/CSS/JS等文件作为嵌入资源编译到项目中，这样操纵既可以加速软件界面的加载速度，也可以避免界面遭到恶意修改。

下面的文章将详细介绍如何将HTML/CSS/JS等文件打包到项目中。

## 第一步 新建NanUI项目
按照之前的教程，新建一个Windows窗体应用程序，并在Program中完成CEF的相关设置操作。

## 第二步 在项目中添加网页资源
在项目中添加index.html文件，内容随意。在本示例中，在页面中添加以下html标记：
```html
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
若要将index.html文件作为嵌入资源编译到项目中，您需要在Visual Studio的**资源管理器**中选中index.html文件，然后在该文件的**属性窗口**的**生成操作**栏目中选中该文件为**嵌入的资源**。如图所示：

![这里写图片描述](http://img.blog.csdn.net/20171226231130613?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGlueHVhbmNoZW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这样，在项目编译时index.html文件将会作为内嵌的资源编译到输出的可执行文件中。

## 第三步 注册嵌入资源的程序集
要让NanUI加载嵌入的资源，需要在项目的Bootstrap执行Load方法成功后注册程序集中的嵌入资源，如下面代码所示，在Load成功后使用**Bootstrap**静态类的**RegisterAssemblyResources**将当前程序集中的资源注册到环境中。
```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace EmbeddedHtmlResources
{
	using NetDimension.NanUI;

	static class Program
	{

		/// <summary>
		/// 应用程序的主入口点。
		/// </summary>
		[STAThread]
		static void Main()
		{
			Application.EnableVisualStyles();
			Application.SetCompatibleTextRenderingDefault(false);

			//指定CEF架构和文件目录结构，并初始化CEF
			if (Bootstrap.Load(PlatformArch.Auto, System.IO.Path.Combine(Application.StartupPath, "fx"), System.IO.Path.Combine(Application.StartupPath, "fx\\Resources"), System.IO.Path.Combine(Application.StartupPath, "fx\\Resources\\locales")))
			{
				//注册嵌入资源，并为指定资源指定一个假的域名my.resource.local
				Bootstrap.RegisterAssemblyResources(System.Reflection.Assembly.GetExecutingAssembly(), "my.resource.local");

				Application.Run(new Form1());
			}

		}
	}
}
```
RegisterAssemblyResources方法的第一个参数是需要执行注册的程序集实例，第二个参数指定一个虚假的域名，嵌入的资源将通过该域名进行访问。

除了将html/css/js文件打包到当前项目，还可以将这些资源打包到单独的dll中，然后用反射的方式来获取该资源dll的程序集并使用上述方法进行注册。例如，将另一个index.html文件打包到EmbeddedResourcesInSplitAssembly.dll程序集中，内容可以与上面的本地项目中的index.html有所差别，然后主项目在CEF初始化成功之后通过下面的方法，也可以将外部程序集中的资源加载到NanUI环境中的：
```C#
//指定CEF架构和文件目录结构，并初始化CEF
			if (Bootstrap.Load(PlatformArch.Auto, System.IO.Path.Combine(Application.StartupPath, "fx"), System.IO.Path.Combine(Application.StartupPath, "fx\\Resources"), System.IO.Path.Combine(Application.StartupPath, "fx\\Resources\\locales")))
			{
				//注册嵌入资源，并为指定资源指定一个假的域名my.resource.local
				Bootstrap.RegisterAssemblyResources(System.Reflection.Assembly.GetExecutingAssembly(), "my.resource.local");
				//加载分离式的资源
				var separateAssembly = System.Reflection.Assembly.LoadFile(System.IO.Path.Combine(Application.StartupPath, "EmbeddedResourcesInSplitAssembly.dll"));
				//注册外部的嵌入资源，并为指定资源指定一个假的域名separate.resource.local
				Bootstrap.RegisterAssemblyResources(separateAssembly , "separate.resource.local");

				Application.Run(new Form1());
			}
```

您可以在项目中注册多个程序中的嵌入式资源，只需指定不同的域名(*domin参数*)即可。

## 第四步 在Formium窗体中使用嵌入的资源
按照上述方法，我们就完成了资源文件的嵌入。那么如何这些嵌入的资源文件呢？其实就跟平常加载网页上的资源是一样的，只需要按照我们指定的虚假域名和资源嵌入的路径组合以后，就可以在NanUI中使用嵌入的资源了。

例如，按照上面步骤中的方式对文件进行嵌入后，我们就可以通过http://my.resource.local/index.html访问到主项目中的index.html文件，然后通过 http://separate.resource.local/index.html访问到外部程序集EmbeddedResourcesInSplitAssembly.dll中的index.html文件了。

那么，如何测试我们的资源是否这确被嵌入到程序集，并且也正常加载了呢？最简单的例子就是在我们的窗体中直接使用这些地址。
```C#
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace EmbeddedHtmlResources
{
	using NetDimension.NanUI;

	public partial class Form1 : Formium
	{
		public Form1() 
			: base("http://my.resource.local/index.html")
		{
			InitializeComponent();
		}
	}
}
```
编译并执行项目文件，我们就可以看到嵌入的资源已经可以正确加载了。
![这里写图片描述](http://img.blog.csdn.net/20171226234319349?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGlueHVhbmNoZW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 示例源码
```
git clone https://github.com/NetDimension/NanUI-Examples-02-Use-Embedded-Resources-In-NanUI.git
```

## 社群和帮助
**GitHub**
[https://github.com/NetDimension/NanUI/](https://github.com/NetDimension/NanUI/)

**交流群QQ群**
521854872

**赞助作者**

如果你喜欢我的工作，并且希望NanUI持续的发展，请对NanUI项目进行捐助以此来鼓励和支持我继续NanUI的开发工作。你可以使用**微信**或者**支付宝**来扫描下面的二维码进行捐助。

![Screen Shot](http://ohtrip.cn/media/beg_with_border.png)

