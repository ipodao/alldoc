# 1. Libgdx介绍与工程搭建

例子可以运行在以下平台：

* Windows
* Linux
* Mac OS X
* Android (Version 1.5 and above)
* HTML5 (Using JavaScript and WebGL)

Libgdx是一游戏框架而不是一个游戏引擎。

## （未整理）1.2 Libgdx 0.97特性

## 1.3 社区

- 官方论坛：http://badlogicgames.com/forum/。
- Blog: http://www.badlogicgames.com/
- Wiki: http://code.google.com/p/libgdx/wiki/TableOfContents
- API overview: http://libgdx.badlogicgames.com/nightlies/docs/api/

## 1.5 创建新应用

常常需要在Eclipse中创建几个工程：一个用于公共代码，一个用于桌面启动，一个用于Android，一个用于HTML5/GWT等。

幸运的是，*Libgdx Project Setup*可以产生这些工程，你只需要导入Eclipse。

运行`gdx-setup-ui`。

![](gdx-setup-ui.png)

工程名输入`demo`。包名`com.packtpub.libgdx.demo`。

In another box called LIBRARY SELECTION, the status of required libraries is 
shown. If there is any item listed in red color, it needs to be fixed before any project 
can be generated. You should see LibGDXbeing listed in red in the Required
section. Click on the blue folder icon next to it as shown in the following screenshot

![](create-project.png)

Then, choose the downloaded archive file libgdx-0.9.7.zipfrom C:\libgdx\ and click on Open.

The text color of the LibGDX label should have changed from red to green by now.

产生后，在Eclipse里导入存在的工程。

The second issue requires you to click on the Problemstab in Eclipse. Open the 
Errorslist and right-click on the reported problem, which should say The GWT SDK 
JAR gwt-servlet.jar is missing in the WEB-INF/lib directory. Choose Quick Fix
from the context menu as shown in the following screenshot:

In the Quick Fix dialog, select Synchronize <WAR>/WEB-INF/lib with SDK 
librariesas the desired fix and click on the Finishbutton, as shown in the 
following screenshot:

![](fix_gwt.png)