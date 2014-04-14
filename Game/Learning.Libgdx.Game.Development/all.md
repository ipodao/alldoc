[toc]

## 2 跨平台开发：构建一次，四处部署

接下来将学习 Libgdx 框架的以下组件：
- Backends
- Modules
- Application Life-Cycle and Interface
- Starter Classes

### （未整理）2.1 demo 应用

解释了工程结构。

### 2.2 Backends

Libgdx makes use of several other libraries to interface the specifics of each platform in order to provide cross-platform support for your applications. Generally, a backend is what enables Libgdx to access the corresponding platform functionalities when one of the abstracted (platform-independent) Libgdx methods is called. For example, drawing an image to the upper-left corner of the screen, playing a sound file at a volume of 80 percent, or reading and writing from/to a file.

Libgdx 目前支持下面三种后端：
- LWJGL (Lightweight Java Game Library)
- Android
- JavaScript/WebGL

将来还会有一个 iOS backend。

#### LWJGL (Lightweight Java Game Library)

LWJGL(Lightweight Java Game Library) is an open source Java library. Libgdx用它支持桌面，包括Windows, Linux, and Mac OS X。

网站：http://www.lwjgl.org/。


#### WebGL

WebGL support is one of the latest additions to the Libgdx framework. This backend uses the GWT totranslate Java code into JavaScript and SoundManager2(SM2), among others, to add a combined support for HTML5, WebGL, and audio playback. Note that this backend requires a WebGL-capable web browser to run the application.

You might want to check out the official website of SM2:
http://www.schillmania.com/projects/soundmanager2/.

You might want to check out the official website of WebGL:
http://www.khronos.org/webgl/.

There is also a list of unresolved issues you might want to check out at
 https://github.com/libgdx/libgdx/blob/master/backends/gdx-backendsgwt/issues.txt.

### 2.3 模块

Libgdx提供6个模块。通过`Gdx`类的静态域访问这几个模块。

Libgdx允许你为不同平台设置不同的代码路径（code paths）。例如可以在PC上增加the level of detail，因为PC的计算能力更强。

#### 2.3.1 Application 模块

通过`Gdx.app`访问此模块。It gives you access to the logging facility, a method to shutdown gracefully, persist data, query the Android API version, query the platform type, and query the memory usage.

##### 2.3.1.1 日志


Libgdx 有自己的日志工具。可以设置级别。默认呢是`LOG_INFO`。可以通过配置文件设置，或在运行时动态设置：
```java
Gdx.app.setLogLevel(Application.LOG_DEBUG);
```

可以级别：
* LOG_NONE: This prints nologs. The logging is completely disabled.
* LOG_ERROR: Thisprints error logs only.
* LOG_INFO: This prints error and info logs.
* LOG_DEBUG: This prints error, info, and debug logs.

写日志：
```java
Gdx.app.log("MyDemoTag", "This is an info log.");
Gdx.app.debug("MyDemoTag", "This is a debug log.");
Gdx.app.error("MyDemoTag", "This is an error log.");
```

##### 2.3.1.2 优雅关闭

令应用关闭：
```
Gdx.app.exit();
```

The framework will then stop the execution in the correct order as soon as possible and completely de-allocate any memory that is still in use, freeing both Java and the native heap.

一定要关闭。

##### 2.3.1.3 持久化数据

利用`Preferences`类，存储键值对到文件。

```java
Preferences prefs = Gdx.app.getPreferences("settings.prefs");
```

写入值：
```java
prefs.putInteger("sound_volume", 100); // volume @ 100%
prefs.flush();
```

记得要调用`flush()`。

> 写入文件需要很多时间。尽可能一次性设置完所有值，最终调用一次`flush()`。

读取：

```java
int soundVolume = prefs.getInteger("sound_volume", 50);
```

##### 2.3.1.4 查询 Android API Level

```java
Gdx.app.getVersion();
```

##### 2.3.1.5 查询平台类型

```java
switch (Gdx.app.getType()) {
case Desktop:
	// Code for Desktop application
	break;
case Android:
	// Code for Android application
	break;
case WebGL:
	// Code for WebGL application
	break;
default:
	// Unhandled (new?) platform application
	break;
}
```

##### 2.3.1.6 查询内存使用

```java
long memUsageJavaHeap = Gdx.app.getJavaHeap();
long memUsageNativeHeap = Gdx.app.getNativeHeap();
```

#### 2.3.2 图形模块

通过`Gdx.getGraphics()`或`Gdx.graphics`获取模块。

##### 查询 delta time

当前时间与上一帧的时间差：`Gdx.graphics.getDeltaTime()`。单位秒。

##### 查询屏幕尺寸

`Gdx.graphics.getWidth()`和`Gdx.graphics.getHeight()`。单位像素。

##### 查询 FPS 计数器

Query a built-in frame counter provided by Libgdx to find the average number of frames per second by calling `Gdx.graphics.getFramesPerSecond()`.

#### 2.3.3 Audio 模块

`Gdx.getAudio()`或`Gdx.audio`。

##### 播放声音

加载声音：`Gdx.audio.newSound()`。支持格式：WAV, MP3, and OGG。

解码后的音频数据大小上限是1 MB。

##### 音乐流

To stream music for playback, call `Gdx.audio.newMusic(`).

支持格式：WAV, MP3, and OGG。

#### 2.3.4 输入模块

The input module can be accessed either through `Gdx.getInput()` or by using the shortcut variable `Gdx.input`.

In order to receive andhandle input properly, you should always implement the `InputProcessor` interface and set it as the global handler for input in Libgdx by calling `Gdx.input.setInputProcessor()`.

##### 键盘、触摸、鼠标

Query the system for the last x or y coordinate in the screen coordinates where the screen origin is at the top-left corner by calling either `Gdx.input.getX()`or `Gdx.input.getY()`.
* 判断屏幕是否被触摸或鼠标点击：`Gdx.input.isTouched()`。
* 判断鼠标按钮是否按下：`Gdx.input.isButtonPressed()`。
* 判断键盘是否按下：`Gdx.input.isKeyPressed()`。

##### 加速度计

Query theaccelerometer for its value on the x axis by calling `Gdx.input.getAccelerometerX()`. Replace the Xin the method's name with Yor Zto query the other two axes. Be aware that there will be no accelerometer present on a desktop, so Libgdx always returns 0.

##### 启动和取消震动

让Android设备震动：`Gdx.input.vibrate()`。

运行中的震动可被取消：`Gdx.input.cancelVibrate()`。


##### Catching Android soft keys

You might want to catch Android's soft keys to add an extra handling code for them. 若想捕获后退按钮：`Gdx.input.setCatchBackKey(true)`。若想捕获菜单按钮：`Gdx.input.setCatchMenuKey(true)`。

On a desktop where you have a mouse pointer, you can tell Libgdx to catch it so that you get a permanent mouse input without having the mouse ever leave the application window. To catch the mouse cursor, call `Gdx.input.setCursorCatched(true)`.

#### 2.3.5 文件模块

The filesmodule can be accessed either through `Gdx.getFiles()` or by using the shortcut variable `Gdx.files`.

##### 引用内部文件

You can get a filehandle for an internal file by calling `Gdx.files.internal()`. An internal file is relative to the `assets` folder on the Android and WebGL platforms. On a desktop, it is relative to the `root` folder of the application.

##### 引用外部文件

You can geta file handle for an external file by calling `Gdx.files.external()`. An external file is relative to the *SD card* on the Android platform. On a desktop, it is relative to the *user's home* folder. Note that this is **not available** for WebGL applications.

#### 2.3.6 网络模块

The network module can be accessed either through `Gdx.getNet()` r byusing the shortcut variable `Gdx.net`.

##### HTTP GET and HTTP POST

You can make HTTP GET and POST requests by calling either `Gdx.net.httpGet()` or `Gdx.net.httpPost()`.

##### Client/server sockets

You can create client/server sockets by calling either `Gdx.net.newClientSocket()` or `Gdx.net.newServerSocket()`.

##### Opening a URI in a web browser

To open a Uniform Resource Identifier(URI) in the default web browser, call `Gdx.net.openURI(URI)`.

### 2.4 Libgdx 应用的生命周期和接口

系统状态：create, resize, render, pause, resume, and dispose.


`ApplicationListener`接口有6个方法，对应每种系统状态。
```java
public interface ApplicationListener {
	public void create ();
	public void resize (int width, int height);
	public void render ();
	public void pause ();
	public void resume ();
	public void dispose ();
}
```

![](app_life.png)


在`create()`中初始化。如加载资源，初始化游戏状态。接下来是`resize()`. This is the first opportunity for an application to adjust itself to the available display size (width and height) given in pixels.

Next, Libgdx will handle system events. If no event has occurred in the meanwhile, it is assumed that the application is (still) running. The next state would be render(). This is where a game application will mainly do two things:
- 更新游戏模型
- 将更新后的模型绘制在屏幕上

Afterwards, a decision is made upon which the platform type is detected by Libgdx. On a desktop or in a web browser, the displaying application window can be resized virtually at any time. Libgdx compares the last and current sizes on every cycle so that resize() is only called if the display size has changed. This makes sure that the running application is able to accommodate a changed display size.

Another system event that can occur during runtime is the exitevent. When it occurs, Libgdx will first change to the `pause()` state, which is a very good place to save any data that would be lost otherwise after the application has terminated. Subsequently, Libgdx changes to the dispose() state where an application should do its final clean-up to free all the resources that it is still using.

This is also almost true for *Android*, except that pause() is an intermediate state that is not directly followed by a dispose() state at first. Be aware that this event may occur anytime during application runtime while the user has pressed the Home button or if there is an incoming phone call in the meanwhile. In fact, as long as the Android operating system does not need the occupied memory of the paused application, its state will not be changed to dispose(). Moreover, it is possible that a paused application might receive a resumesystem event, which in this case would change its state to resume(), and it would eventually arrive at the system event handler again.

### 2.5 Starter 类

Starter 类是应用入口。需要为不同平台专门编写，为特定平台指定启动序列。一般该类非常只有几句代码构成，初始化平台特定的配置。启动完成后，Libgdx 框架从 Starter 类接管控制，进入跨平台共享代码，即`ApplicationListener`的实现（如`MyDemo`）。`MyDemo`是共享代码开始的地方。

#### 在桌面运行

The Starter Class forthe desktop application is called `Main.java`.

The following listing is `Main.java` from demo-desktop:

```java
package com.packtpub.libgdx.demo;
import com.badlogic.gdx.backends.lwjgl.LwjglApplication;
import com.badlogic.gdx.backends.lwjgl.LwjglApplicationConfiguration;
public class Main {
	public static void main(String[] args) {
		LwjglApplicationConfiguration cfg =
			new LwjglApplicationConfiguration();
		cfg.title = "demo";
		cfg.useGL20 = false;
		cfg.width = 480;
		cfg.height = 320;
		new LwjglApplication(new MyDemo(), cfg);
	}
}
```

This is all youn eed to write and configure a Starter Class for a desktop.

#### 在Android上运行

The Starter Class for the Android applicationis called `MainActivity.java`.

The following listing is `MainActivity.java` from demo-android:

```java
	package com.packtpub.libgdx.demo;
	import android.os.Bundle;
	import com.badlogic.gdx.backends.android.AndroidApplication;
	import com.badlogic.gdx.backends.android.AndroidApplicationConfiguration;
	public class MainActivity extends AndroidApplication {
		@Override
		public void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			AndroidApplicationConfiguration cfg =
				new AndroidApplicationConfiguration();
			cfg.useGL20 = false;
			initialize(new MyDemo(), cfg);
		}
	}
```

Activity继承自`AndroidApplication`。

So when should GLES2 be used on Android? A better question to ask would be whether you plan to use shaders in your application. If this is the case, opt for GLES2. In any other case, there will be no real benefit except being able to use non-power-of-two textures (also known as NPOT textures); arbitrarily-sized textures that do not equal to widths or heightsrepresentable by the formula `2^n`, such as 32 x 32, 512 x 512, and 128 x 1024.

> NPOT textures are not guaranteed to work on all devices. For example, the Nexus One ignores NPOT textures. Also, they may cause performance penalties on some hardware, so it is best to avoid using this feature at all. In Chapter 4, Gathering Resources, you will learn about a technique called *Texture Atlas*. This will allow you to use arbitrarily-sized textures even when not using GLES2.

The following listing is `AndroidManifest.xml` from demo-android:


```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
	package="com.packtpub.libgdx.demo"
	android:versionCode="1"
	android:versionName="1.0" >
	<uses-sdk android:minSdkVersion="5"
		android:targetSdkVersion="17" />
	<application
		android:icon="@drawable/ic_launcher"
		android:label="@string/app_name" >
		<activity
			android:name=".MainActivity"
			android:label="@string/app_name"
			android:screenOrientation="landscape"
			android:configChanges="keyboard|keyboardHidden|orientation|screenSize">
			<intent-filter>
				<action android:name="android.intent.action.MAIN" />
				<category android:name="android.intent.category.LAUNCHER" />
			</intent-filter>
		</activity>
	</application>
</manifest>
```

#### （未）在带有 WebGL 的浏览器上运行

### 2.6 demo 应用代码















