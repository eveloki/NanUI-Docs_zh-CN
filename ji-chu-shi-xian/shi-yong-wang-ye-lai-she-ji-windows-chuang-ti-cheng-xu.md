创建NanUI应用的方法本文不再阐述，具体方法请参看文章目录里的相关文章。本文将介绍NanUI的核心功能，用一张网页铺满整个窗体区域，并将讲述如何使用CSS和HTML来实现对窗体的拖动、最大化、最小化、关闭等操作。

![这里写图片描述](http://img.blog.csdn.net/20171227105003921)

如图所示的界面，整个窗体的样式都是由HTML和CSS等前端技术来呈现的。具体的HTML/CSS/JS代码本文不详细描述，因为这些和NanUI的关联不大，根据实际项目的需要，您可以为您的软件界面设计出更棒的效果。

在示例界面中，我们将主要介绍系统命令菜单部分的最小化、最大化和关闭按钮，以及界面左侧红色的可用作拖动窗体的纵向标题栏的实现。

用前端技术来解析，左侧纵向标题栏其本质是一条宽度固定，高度100%的DIV；命令区域内的最小化、最大化和关闭按钮其本质是三个SPAN标签内嵌套了三个不同的FontAwsome的图标。下面的内容将介绍它们是如何实现对承载窗体状态改变的。

## 如何通过拖拽HTML元素来移动窗体位置

如果您需要实现类似示例中拖动左侧红色区域标题栏来改变窗体位置的功能，那么在该元素对应的CSS中只需指定该元素所在区域的**-webkit-app-region**属性值为**drag**即可实现：
```CSS
.some-class{
	-webkit-app-region:drag;
}
```
指定此样式后，只要鼠标指针在该样式作用的区域内执行拖拽操作，那么NanUI的承载窗体的位置将随鼠标的拖动而发生改变。

但在设计界面时，如果您希望在可拖动元素区域内的某些区域不接收拖动信号，那么只需要将**-webkit-app-region**属性值改为**no-drag**即可。

例如示例中，三个系统命令按钮实际上是包括在一个DIV元素内，同时这个DIV设置了**-webkit-app-region**属性值为**drag**，这时您会发现，拖动这个DIV所在区域（包括三个命令按钮的区域）时窗体都实现了移动，但是这也阻断了这个区域内的其他鼠标操作，包括三个命令按钮的鼠标点击操作。这明显不是所期望的效果。如下，该区域的HTML代码为：
```HTML
<div class="app-sys-command-container">
	<ul class="sys-commands">
		<li n-ui-command="minimize">
			<i class="fa fa-window-minimize"></i>
		</li>
		<li n-ui-command="maximize">
			<i class="fa fa-window-maximize"></i>
		</li>
		<li n-ui-command="close">
			<i class="fa fa-close"></i>
		</li>
	</ul>
</div>
```
在css样式.app-sys-command-container中设置了**-webkit-app-region**属性值为**drag**，这导致了ul.sys-commands区域也呈现可拖动的状态。为了避免这个区域被默认的拖动事件影响到其他事件的相应，那么就需要设置.sys-commands的样式**-webkit-app-region**属性值为**no-drag**，那么这部分区域将不再相应窗体拖动的事件。这部分的CSS代码为：
```css
.demo-app > content > .app-sys-command-container {
	...
	-webkit-app-region: drag;
	...
}

.demo-app > content > .app-sys-command-container > .sys-commands {
	...
	-webkit-app-region: no-drag;
	...
}
```
这样，您就可以灵活的控制可拖动来改变窗体位置的区域了。

## 如何通过HTML元素来执行窗体的最大化/最小化/关闭操作
在上面的html代码片段中，展示了示例程序的三个系统命令按钮的实现方式：
```HTML
<div class="app-sys-command-container">
	<ul class="sys-commands">
		<li n-ui-command="minimize">
			<i class="fa fa-window-minimize"></i>
		</li>
		<li n-ui-command="maximize">
			<i class="fa fa-window-maximize"></i>
		</li>
		<li n-ui-command="close">
			<i class="fa fa-close"></i>
		</li>
	</ul>
</div>
```
其中的i标签内，可以看到不常见的html属性**n-ui-command**，这一属性是NanUI用来标注按钮行为的专用属性，通过其属性值minimize/maximize/close不难看出，通过点击具备这一专用属性的标签，就能够实现窗体对应的最小化/最大化/关闭操作。

特别需要指出的，在示例窗体中点击最大化按钮后可以看到，最大化按钮的图标从**最大化**改变成了**还原**的样式，这是通过使用Javascript监测窗体事件来实现的。

如同上面介绍的专用属性，NanUI还内置了一些专用的事件。通过监听这些事件来实现一些特殊的功能，例如上面所说的最大化窗体时改变系统按钮的图标，又或者是窗体市区焦点时改变标题栏的颜色等。

## NanUI 窗体专用事件
目前NanUI实现的专用事件有以下三个：

- **hoststatechange**: 窗体状态（最大化、最小化、还原）发生改变时触发。
- **hostactivestate**: 窗体获得或丢失焦点时触发。
- **hostsizechange**: 窗体大小改变时触发。

通过监听这些事件，您就可以根据需求来实现一些特定的效果。如示例项目中，使用了jQuery来监听这三个专用事件：
```js
$(function () {
	//窗口状态改变
	$(window).on("hoststatechange", function (e) {
		console.log(e.detail);
		$("#label-form-state").text(e.detail.stateName);

	});
	
	//窗口激活状态改变
	$(window).on("hostactivestate", function (e) {
		console.log(e.detail);
		$("#label-activated-state").text(e.detail.stateName);

		if (e.detail) {
			if (e.detail.state === 1) {
				$("html").addClass("app-active");
			}
			else {
				$("html").removeClass("app-active");
			}
		}
	});
	//窗口尺寸改变
	$(window).on("hostsizechange", function (e) {
		console.log(e.detail);
		$("#label-size-state").text(`width:${e.detail.width} height:${e.detail.height}`);

	});

});
```

上述代码的具体效果，可以自行对示例程序进行调试来查看效果。


## 内置Javascript对象 - NanUI
NanUI除了实现了上述的专用html属性n-ui-command和三个专用事件外，在Javascript全局环境中还内置了一个专用对象**NanUI**，通过该对象您可以查询当前NanUI和CEF的版本号，通过hostWindow中的方法来获取当前窗体的状态值，执行最大化、最小化和关闭操作。

![这里写图片描述](http://img.blog.csdn.net/20171227114228407?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGlueHVhbmNoZW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 示例源码
```
git clone https://github.com/NetDimension/NanUI-Examples-03-Design-Your-Form-With-Html.git
```

## 社群和帮助
**GitHub**
[https://github.com/NetDimension/NanUI/](https://github.com/NetDimension/NanUI/)

**交流群QQ群**
521854872

**赞助作者**

如果你喜欢我的工作，并且希望NanUI持续的发展，请对NanUI项目进行捐助以此来鼓励和支持我继续NanUI的开发工作。你可以使用**微信**或者**支付宝**来扫描下面的二维码进行捐助。

![Screen Shot](http://ohtrip.cn/media/beg_with_border.png)

