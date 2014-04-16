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

分析`MyDemo.java`：

```java
public class MyDemo implements ApplicationListener {
	private OrthographicCamera camera;
	private SpriteBatch batch;
	private Texture texture;
	private Sprite sprite;
}
```

We will use the orthographic camera for displaying our 2D scenes. The camera is the player's view of the actual scene in the game which is defined by a certain width and height (also called *viewport*).

For more information about projections, check out the great article *Orthographic vs. Perspective* by Jeff Lamarche at http://iphonedevelopment.blogspot.de/2009/04/opengl-es-from-ground-up-part-3.html.

The `batch` variable is of the class type `SpriteBatch`. This is where you send all your drawing commands to Libgdx. Beyond the ability of this class to draw images, it is also capable of optimizing the drawing performance under certain circumstances.

The `texture` variable is of the class type `Texture`. It holds a reference to the actual image; the texture data that is stored in memory at runtime.

The `sprite` variable is of the class type `Sprite`. It is a complex data type that contains lots of attributes to represent a graphical object that has a position in 2D space, width, and height. It can also be rotated and scaled. Internally, it holds a reference to a `TextureRegion` class that in turn is a means to cut out a certain portion of a texture.

Now that we have a basic knowledge of the involved data types, we can advance to the implementation details of the `ApplicationListener` interface.

In the MyDemo class, the only methods containing code are `create()`, `render()`, and `dispose()`. The remaining three methods are left empty, which is just fine.

#### create() 方法

```java
	@Override
	public void create() {
		float w = Gdx.graphics.getWidth();
		float h = Gdx.graphics.getHeight();
		camera = new OrthographicCamera(1, h / w);
		batch = new SpriteBatch();
		texture = new Texture(Gdx.files.internal("data/libgdx.png"));
		texture.setFilter(TextureFilter.Linear, TextureFilter.Linear);
		TextureRegion region = new TextureRegion(texture, 0, 0, 512, 275);
		sprite = new Sprite(region);
		sprite.setSize(0.9f, 0.9f * sprite.getHeight() / sprite.getWidth());
		sprite.setOrigin(sprite.getWidth() / 2, sprite.getHeight() / 2);
		sprite.setPosition(-sprite.getWidth() / 2, -sprite.getHeight() / 2);
	}
```

Then a new instance of `SpriteBatch` is created so that images can be drawn and made visible with the camera.

In order to be able to use the filled part of this image only, a new instance of `TextureRegion` is created. It references the previously loaded texture that contains the full image and has the additional information to cut out all the pixels starting from (0, 0) to (512, 275). These two points describe a rectangle starting at the top-left corner of the image with a width and height of 512 by 275 pixels. Finally, a sprite is created using the information of the previously created texture region. The sprite's size is set to 90 percent of its original size. 精灵的原点设在屏幕中央。然后设置相对位置，反向平移半个精灵大小，这样精灵就能完全居中了。

> Libgdx 的坐标原点位于屏幕左下角。This means that the positive x axis points right while the positive y axis points up.

#### render() 方法

```java
@Override
public void render() {
	Gdx.gl.glClearColor(1, 1, 1, 1);
	Gdx.gl.glClear(GL10.GL_COLOR_BUFFER_BIT);
	batch.setProjectionMatrix(camera.combined);
	batch.begin();
	sprite.draw(batch);
	batch.end();
}
```

The first two lines call low-level OpenGL methods to set the clear color to a solid white and then execute the clear screen command.

Next, the projection matrix of the sprite batch is set to the camera's combined projection and view matrix. You do not have to understand what this means in detail at the moment. It basically just means that every following drawing command will behave to the rules of an orthographic projection, or simply spoken drawing will be done in 2D space using the position and bounds of the given camera.

`begin()` 和 `end()` 总应该成对出现，且不能有嵌套，否则会出错。The actual drawing of the sprite is accomplished by calling the `draw()` method of the sprite to draw and pass the instance of the sprite batch.

#### dispose() 方法

This is the place where you should clean up and free all resources that are still in use by an application:

```java
@Override
public void dispose() {
	batch.dispose();
	texture.dispose();
}
```

每个需要分配资源（及内存）的 Libgdx 类都会实现 `Disposable` 接口。可以通过 `dispose()` 方法解除分配。

完整代码
```java
package com.packtpub.libgdx.demo;

import com.badlogic.gdx.ApplicationListener;
import com.badlogic.gdx.Gdx;
import com.badlogic.gdx.graphics.GL10;
import com.badlogic.gdx.graphics.OrthographicCamera;
import com.badlogic.gdx.graphics.Texture;
import com.badlogic.gdx.graphics.Texture.TextureFilter;
import com.badlogic.gdx.graphics.g2d.Sprite;
import com.badlogic.gdx.graphics.g2d.SpriteBatch;
import com.badlogic.gdx.graphics.g2d.TextureRegion;
public class MyDemo implements ApplicationListener {
	private OrthographicCamera camera;
	private SpriteBatch batch;
	private Texture texture;
	private Sprite sprite;
	@Override
	public void create() {
	float w = Gdx.graphics.getWidth();
	float h = Gdx.graphics.getHeight();
	camera = new OrthographicCamera(1, h / w);
	batch = new SpriteBatch();
	texture = new Texture(Gdx.files.internal("data/libgdx.png"));
	texture.setFilter(TextureFilter.Linear, TextureFilter.Linear);
	TextureRegion region = new TextureRegion(texture, 0, 0, 512, 275);
	sprite = new Sprite(region);
	sprite.setSize(0.9f, 0.9f * sprite.getHeight() / sprite.getWidth());
	sprite.setOrigin(sprite.getWidth() / 2, sprite.getHeight() / 2);
	sprite.setPosition(-sprite.getWidth() / 2, -sprite.getHeight() / 2);
	}
	@Override
	public void dispose() {
		batch.dispose();
		texture.dispose();
	}
	@Override
	public void render() {
		Gdx.gl.glClearColor(1, 1, 1, 1);
		Gdx.gl.glClear(GL10.GL_COLOR_BUFFER_BIT);
		batch.setProjectionMatrix(camera.combined);
		batch.begin();
		sprite.draw(batch);
		batch.end();
	}
	@Override
	public void resize(int width, int height) {
	}
	@Override
	public void pause() {
	}
	@Override
	public void resume() {
	}
}
```

#### 旋转

持续旋转：
```java
private float rot;
@Override
public void render() {
	Gdx.gl.glClearColor(1, 1, 1, 1);
	Gdx.gl.glClear(GL10.GL_COLOR_BUFFER_BIT);
	batch.setProjectionMatrix(camera.combined);
	batch.begin();
	final float degressPerSecond = 10.0f;
	rot = (rot + Gdx.graphics.getDeltaTime() * degressPerSecond) % 360;
	sprite.setRotation(rot);
	sprite.draw(batch);
	batch.end();
}
```

变量`rot`跟踪当前角度。

Since the Sine (or Cosine) function has an oscillating behavior, we can make perfect use of it to make the image shake by a certain amount to the left and right. The amount (amplitude) can be increased and decreased by multiplying it with the answer of the Sine function.

```java
@Override
public void render() {
	Gdx.gl.glClearColor(1, 1, 1, 1);
	Gdx.gl.glClear(GL10.GL_COLOR_BUFFER_BIT);
	batch.setProjectionMatrix(camera.combined);
	batch.begin();
	float degressPerSecond = 10.0f;
	rot = (rot + Gdx.graphics.getDeltaTime() * degressPerSecond) % 360;
	final float shakeAmplitudeInDegrees = 5.0f;
	float shake = MathUtils.sin(rot) * shakeAmplitudeInDegrees;
	sprite.setRotation(shake);
	sprite.draw(batch);
	batch.end();
}
```

## 3 配置游戏

本章开始构建游戏 **Canyon Bunny**。

### 3.1 建立 Canyon Bunny 工程

Run the **gdx-setup-ui** tool from Libgdx and use the followingsettings:
* Name: CanyonBunny
* Package: com.packtpub.libgdx.canyonbunny
* Game class: CanyonBunnyMain
* Destination: C:\libgdx
* Generate the desktop project: select the checkbox
* Generate the html project: select the checkbox

产生四个工程：CanyonBunny, CanyonBunny-desktop, CanyonBunny-android, and CanyonBunny-html.

打开`CanyonBunny-android/res/strings.xml`，修改应用名：
```xml
<string name="app_name">Canyon Bunny</string>
```

移除以下目录和文件：
* CanyonBunny/src/com/packtpub/libgdx/canyonbunny/CanyonBunnyMain.java
* CanyonBunny-android/assets/data/libgdx.png
* CanyonBunny-android/assets/data/

Then, open `CanyonBunny-desktop/com/packtpub/libgdx/canyonbunny/Main.java` and change the resolution parameters for width and height to 800 x 480 pixels like this:
```java
cfg.width = 800;
cfg.height = 480;
```

### 3.2 UML

![](CanyonBunny_UML.png)

The `Assets` class that will be used to organize and simplify the way to access the game's assets.

The `WorldController` class contains all the game logic to initialize and modify the game world. It also needs access to `CameraHelper`, a helper class for the camera that, for example, enables it to target and follow any game object; `Level` that holds the level data; and a list of `AbstractGameObject` instances representing any game object that exists in the game world.

The rendering takes place in `WorldRenderer` that apparently also requires it to have access to the list of `AbstractGameObject` instances. Since the game objects need to be created before the process of modification and rendering, `Level` needs access to the list of `AbstractGameObject` instances as well when a level is loaded from a level file at the beginning of the game.

最下面一行都是`AbstractGameObject`的子类，它们都可以被渲染到游戏中。这些类按照它们的目的被分组：
- 玩家角色
  - BunnyHead：表示玩家控制的角色
- Level Objects
  - Rock: 表示一个平台，有左右边界，中间部分可以设置为任意长度。It is the ground in a level where the player will move on.
- Level Items
  - GoldCoin: 拾起后增加玩家分数。
  - Feather：拾起后可以让玩家飞行。
- Level Decorations
  - WaterOverlay: It represents an image that is attached to the camera's horizontal position，因此不管摄像头在x轴如何移动，它总是可见的。
  - Mountains: 两个山的图像，以不同的速度移动，以模拟视差幻觉（parallax optical illusion）
  - Cloud: 云，向左边慢慢移动。

### 3.3 类主框架

本节先介绍几个主类的骨架，实现在下一节。

#### 实现 `Constants`

用`Constants`存储常量。

Here is a listing of the code for Constants:
```java
package com.packtpub.libgdx.canyonbunny.util;
public class Constants {
	// Visible game world is 5 meters wide
	public static final float VIEWPORT_WIDTH = 5.0f;
	// Visible game world is 5 meters tall
	public static final float VIEWPORT_HEIGHT = 5.0f;
}
```

First, we need to define the visible world size that can be seen at once when not moving around in the game world.

#### 实现 `CanyonBunnyMain`

`CanyonBunnyMain`实现了`ApplicationListener`，于是可以成为 starter 类。

```java
package com.packtpub.libgdx.canyonbunny;
import com.badlogic.gdx.ApplicationListener;
import com.packtpub.libgdx.canyonbunny.game.WorldController;
import com.packtpub.libgdx.canyonbunny.game.WorldRenderer;
public class CanyonBunnyMain implements ApplicationListener {
	private static final String TAG = CanyonBunnyMain.class.getName();
	private WorldController worldController;
	private WorldRenderer worldRenderer;
	@Override public void create () { }
	@Override public void render () { }
	@Override public void resize (int width, int height) { }
	@Override public void pause () { }
	@Override public void resume () { }
	@Override public void dispose () { }
}
```

A reference each to `WorldController` and `WorldRenderer` enables this class to update and control the game's flow and also to render the game's current state to the screen.

#### 实现 `WorldController`

```java
package com.packtpub.libgdx.canyonbunny.game;
public class WorldController {
	private static final String TAG = WorldController.class.getName();
	public WorldController () { }
	private void init () { }
	public void update (float deltaTime) { }
}
```

`update()`方法每秒会被调用几百次。It requires a delta time so that it can apply updates to the game world according to the fraction of time that has passed since the last rendered frame.

> The configurations of our starter classes use **vertical synchronization (vsync)**, which is enabled by default. Using vsync will cap your frame rate and likewise the calls to `update()` at a maximum of 60 frames per second.

#### 实现 `WorldRenderer`

```java
package com.packtpub.libgdx.canyonbunny.game;
import com.badlogic.gdx.graphics.OrthographicCamera;
import com.badlogic.gdx.graphics.g2d.SpriteBatch;
import com.badlogic.gdx.utils.Disposable;
import com.packtpub.libgdx.canyonbunny.util.Constants;
public class WorldRenderer implements Disposable {
	private OrthographicCamera camera;
	private SpriteBatch batch;
	private WorldController worldController;
	public WorldRenderer (WorldController worldController) { }
	private void init () { }
	public void render () { }
	public void resize (int width, int height) { }
	@Override public void dispose () { }
}
```

Furthermore, it contains a `render()` method that will contain the logic to define in which order game objects are drawn over others. Whenever the screen size is changed, including the event at the start of the program, `resize()` will spring into action and initiate the required steps to accommodate the new situation.

The rendering is accomplished by using an orthographic camera that is suitable for two-dimensional projections. Fortunately, Libgdx comes with a ready-to-use `OrthographicCamera` class to simplify our 2D rendering tasks. The `SpriteBatch` class is the actual workhorse that draws all our objects with respect to the camera's current settings (for example, position, zoom, and so on) to the screen. Since `SpriteBatch` implements Libgdx's `Disposable` interface, it is advisable to always call its `dispose()` method to free the allocated memory when it is no longer needed. We will do this in `WorldRenderer` by also implementing the `Disposable` interface. This allows us to easily cascade the disposal process when `dispose()` in `CanyonBunnyMain` is called by Libgdx. In this case, we will simply call the `WorldRenderer`'s class' `dispose()` method which in turn will call the `SpriteBatch` class' `dispose()` method.

### 3.4 放到一起

本节实现上一节介绍的几个主类。首先是游戏循环。接着引入一些精灵，验证更新和渲染机制。为了能操控游戏，引入控制器接收和响应用户输入。最后，实现`CameraHelper`类，让用户在游戏中自有移动，and to select a game object of our choice that the camera is supposed to follow.

#### 3.4.1 建立游戏循环

游戏循环位于`CanyonBunnyMain.render()`方法内。

为了Android，需要特殊处理暂停。在暂停时停止绘制，在Resume时恢复。为此，定义变量
```java
	private boolean paused;
```

```java
	@Override
	public void create () {
		// Set Libgdx log level to DEBUG
		Gdx.app.setLogLevel(Application.LOG_DEBUG);
		// 初始化控制器和渲染器
		worldController = new WorldController();
		worldRenderer = new WorldRenderer(worldController);

		// Game world is active on start
		paused = false;
	}

	@Override
	public void pause () {
		paused = true;
	}

	@Override
	public void resume () {
		paused = false;
	}
```

为了能够持续的更新和渲染游戏到屏幕，实现`render()`方法：
```javascript
	@Override
	public void render () {
		// Do not update game world when paused.
		if (!paused) {
			// Update game world by the time that has passed
			// since last rendered frame.
			worldController.update(Gdx.graphics.getDeltaTime());
		}	
		// 设置清屏颜色：Cornflower Blue
		Gdx.gl.glClearColor(0x64/255.0f, 0x95/255.0f, 0xed/255.0f, 	0xff/255.0f);
		// 清屏
		Gdx.gl.glClear(GL10.GL_COLOR_BUFFER_BIT);
		// 渲染
		worldRenderer.render();
	}
```

注意一定要安装上面的顺序执行：先更新、清屏，最后渲染。

The game world is incrementally updated using delta times. Luckily, Libgdx already does the math and housekeeping behind this for us, so all we need to do is to query the value by calling `getDeltaTime()` from the `Gdx.graphics` module and pass it to `update()` of `WorldController`. 

`glClearColor()`取值`(red, green, blue, and alpha)`（RGBA）。每个颜色值是从0到1的浮点数。这里通过将十六进制数除以`255.0f`求得这个浮点数。

`glClear()`清楚屏幕所有内容，并用清屏色填充。

Next, add the following code to resize():

```java
	@Override
	public void resize (int width, int height) {
		worldRenderer.resize(width, height);
	}
```

因为调整大小事件与渲染有关，因此交给`WorldRenderer`处理。


The same is almost true for the code to be added in dispose():

```java
	@Override
	public void dispose() {
		worldRenderer.dispose();
	}
```

此时运行游戏应该显示蓝色背景。


#### 3.4.2 添加测试精灵

向`WorldController`添加代码：

```java
	public Sprite[] testSprites;
	public int selectedSprite;
	public WorldController () {
		init();
	}
	private void init () {
		initTestObjects();
	}
	private void initTestObjects() {
		// 创建 5 个精灵
		testSprites = new Sprite[5];
		// Create empty POT-sized Pixmap
		int width = 32;
		int height = 32;
		Pixmap pixmap = createProceduralPixmap(width, height);
		// Create a new texture from pixmap data
		Texture texture = new Texture(pixmap);
		// 使用刚才创建的纹理创建新精灵
		for (int i = 0; i < testSprites.length; i++) {
			Sprite spr = new Sprite(texture);
			// 将精灵大小设为1m x 1m（游戏世界）
			spr.setSize(1, 1);
			// 设置精灵原点
			spr.setOrigin(spr.getWidth() / 2.0f,
				spr.getHeight() / 2.0f);
			// 计算精灵的随机位置
			float randomX = MathUtils.random(-2.0f, 2.0f);
			float randomY = MathUtils.random(-2.0f, 2.0f);
			spr.setPosition(randomX, randomY);
			// Put new sprite into array
			testSprites[i] = spr;
		}
		// 默认选中第一个精灵
		selectedSprite = 0;
	}
	private Pixmap createProceduralPixmap (int width, int height) {
		Pixmap pixmap = new Pixmap(width, height, Format.RGBA8888);
		// 红色，50%透明
		pixmap.setColor(1, 0, 0, 0.5f);
		pixmap.fill();
		// Draw a yellow-colored X shape on square
		pixmap.setColor(1, 1, 0, 1);
		pixmap.drawLine(0, 0, width, height);
		pixmap.drawLine(width, 0, 0, height);
		// Draw a cyan-colored border around square
		pixmap.setColor(0, 1, 1, 1);
		pixmap.drawRectangle(0, 0, width, height);
		return pixmap;
	}
	public void update (float deltaTime) {
		updateTestObjects(deltaTime);
	}
	private void updateTestObjects(float deltaTime) {
		// Get current rotation from selected sprite
		float rotation = testSprites[selectedSprite].getRotation();
		// 每秒旋转90度
		rotation += 90 * deltaTime;
		// Wrap around at 360 degrees
		rotation %= 360;
		// Set new rotation value to selected sprite
		testSprites[selectedSprite].setRotation(rotation);
	}
}
```

精灵的大小被设为 1*1 米。之前我们将世界定义为 5*5 米。因此该精灵将占据1/5大小。

当目前为止，还只是创建和修改了游戏世界。尚未将其渲染到屏幕。接下来实现渲染。

向`WorldRenderer`添加：
```java
	public WorldRenderer (WorldController worldController) {
		this.worldController = worldController;
		init();
	}
	private void init () {
		batch = new SpriteBatch();
		camera = new OrthographicCamera(Constants.VIEWPORT_WIDTH,
			Constants.VIEWPORT_HEIGHT);
		camera.position.set(0, 0, 0);
		camera.update();
	}
	public void render () {
		renderTestObjects();
	}
	private void renderTestObjects() {
		batch.setProjectionMatrix(camera.combined);
		batch.begin();
		for(Sprite sprite : worldController.testSprites) {
			sprite.draw(batch);
		}
		batch.end();
	}
	public void resize (int width, int height) {
		camera.viewportWidth = (Constants.VIEWPORT_HEIGHT / height) *  width;
		camera.update();
	}
	@Override
	public void dispose () {
		batch.dispose();
	}
```

首先注意到`WorldRenderer`引用`WorldController`，以访问控制器控制的游戏对象。

The camera's viewport defines the size of the captured game world it is looking at. It works basically the same as a real camera. 即，通过摄像机观察世界时，你只能看到镜头中的部分。当你向看左边时，需要向左移动摄像机。

`SpriteBatch`类的`begin()`和`end()`方法用于启动和结束一次批量绘制。

#### 3.4.3 调试辅助

The debug controls we are going to implement will allow us to do the following:
- 移动选中的精灵（上下左右）
- 将游戏世界重置为初始状态
- Cycle through the list of sprites to select the other ones

第一个要求与后两个有很大区别。例如，当按下表示移动的键时，当键被按住时，操作就应该持续进行。下面将介绍两种处理按钮事件的方式。第一种利用`update()`轮询。第二种利用标准事件监听。

Let us begin with the movement of a selected sprite that uses the continuous execution approach. 向`WorldController`增加以下代码：

```java
	public void update (float deltaTime) {
		handleDebugInput(deltaTime);
		updateTestObjects(deltaTime);
	}
	private void handleDebugInput (float deltaTime) {
		if (Gdx.app.getType() != ApplicationType.Desktop) return;
		// Selected Sprite Controls
		float sprMoveSpeed = 5 * deltaTime;
		if (Gdx.input.isKeyPressed(Keys.A))
			moveSelectedSprite(-sprMoveSpeed, 0);
		if (Gdx.input.isKeyPressed(Keys.D))
			moveSelectedSprite(sprMoveSpeed, 0);
		if (Gdx.input.isKeyPressed(Keys.W))
			moveSelectedSprite(0, sprMoveSpeed);
		if (Gdx.input.isKeyPressed(Keys.S))
			moveSelectedSprite(0, -sprMoveSpeed);
	}
	private void moveSelectedSprite (float x, float y) {
		testSprites[selectedSprite].translate(x, y);
	}
```

The next controls to implement are the keys to reset the game world and to select the next sprite.

```java
public class WorldController extends InputAdapter {
	// ...
	private void init () {
		Gdx.input.setInputProcessor(this);
		initTestObjects();
	}
}
```

`InputAdapter`类是`InputListener`接口的适配器类。利用`Gdx.input.setInputProcessor(this);`方法将`WorldController`类设为接收输入事件的类。

Now that Libgdx will send all the input events to our listener, we need to actually implement an event handler for each event we are interested in.

```java
@Override
public boolean keyUp (int keycode) {
	// Reset game world
	if (keycode == Keys.R) {
		init();
		Gdx.app.debug(TAG, "Game world resetted");
	}
	// Select next sprite
	else if (keycode == Keys.SPACE) {
		selectedSprite = (selectedSprite + 1) % testSprites.length;
		Gdx.app.debug(TAG, "Sprite #" + selectedSprite + " selected");
	}
	return false;
}
```

> {{之前处理ADWS按钮事件（处理移动）的方式是周期性的轮询——利用FPS。而上面对R和Space按钮的监听是标准的事件驱动，回调方式。如果事件队列比游戏循环FPS频率更高，响应更快，则回调方式不容易丢事件。}}

#### 3.4.4 CameraHelper

编写一个帮助类`CameraHelper`，用于管理和操纵摄像机。

```java
package com.packtpub.libgdx.canyonbunny.util;
import com.badlogic.gdx.graphics.OrthographicCamera;
import com.badlogic.gdx.graphics.g2d.Sprite;
import com.badlogic.gdx.math.MathUtils;
import com.badlogic.gdx.math.Vector2;
public class CameraHelper {
	private static final String TAG = CameraHelper.class.getName();
	private final float MAX_ZOOM_IN = 0.25f;
	private final float MAX_ZOOM_OUT = 10.0f;
	private Vector2 position;
	private float zoom;
	private Sprite target;
	public CameraHelper () {
		position = new Vector2();
		zoom = 1.0f;
	}
	public void update (float deltaTime) {
		if (!hasTarget()) return;
		position.x = target.getX() + target.getOriginX();
		position.y = target.getY() + target.getOriginY();
	}
	public void setPosition (float x, float y) {
		this.position.set(x, y);
	}
	public Vector2 getPosition () { return position; }
	public void addZoom (float amount) { setZoom(zoom + amount); }
	public void setZoom (float zoom) {
		this.zoom = MathUtils.clamp(zoom, MAX_ZOOM_IN, MAX_ZOOM_OUT);
	}
	public float getZoom () { return zoom; }
	public void setTarget (Sprite target) { this.target = target; }
	public Sprite getTarget () { return target; }
	public boolean hasTarget () { return target != null; }
	public boolean hasTarget (Sprite target) {
		return hasTarget() && this.target.equals(target);
	}
	public void applyTo (OrthographicCamera camera) {
		camera.position.x = position.x;
		camera.position.y = position.y;
		camera.zoom = zoom;
		camera.update();
	}
}
```

把一个游戏对象设为目标后，可以让摄像头追踪它的位置。每个更新循环都要调用`update()`方法，根据目标对象位置更新摄像机位置。The `applyTo()` method should always be called at the beginning of the rendering of a *new frame* as it takes care of updating the camera's attributes.

#### 3.4.5 向`CameraHelper`添加调试控制

允许你自由自动，缩放，跟踪某个游戏对象。

向`WorldController`添加：
```java
public CameraHelper cameraHelper;

private void init () {
	Gdx.input.setInputProcessor(this);
	cameraHelper = new CameraHelper();
	initTestObjects();
}

public void update (float deltaTime) {
	handleDebugInput(deltaTime);
	updateTestObjects(deltaTime);
	cameraHelper.update(deltaTime);
}

@Override
public boolean keyUp (int keycode) {
	// Reset game world
	if (keycode == Keys.R) {
		init();
		Gdx.app.debug(TAG, "Game world resetted");
	}
	// Select next sprite
	else if (keycode == Keys.SPACE) {
		selectedSprite = (selectedSprite + 1) % testSprites.length;
		// Update camera's target to follow the currently
		// selected sprite
		if (cameraHelper.hasTarget()) {
			cameraHelper.setTarget(testSprites[selectedSprite]);
		}
		Gdx.app.debug(TAG, "Sprite #" + selectedSprite + " selected");
	}
	// Toggle camera follow
	else if (keycode == Keys.ENTER) {
		cameraHelper.setTarget(cameraHelper.hasTarget() ? null
			: testSprites[selectedSprite]);
		Gdx.app.debug(TAG, "Camera follow enabled: " + 
			cameraHelper.hasTarget());
	}
	return false;
}
```

Remember to continuously call `update()` of CameraHelperon every update cycle to ensure that its internal calculations are also performed.

Additionally, add the following code to `WorldRenderer`:
```java
public void renderTestObjects () {
	worldController.cameraHelper.applyTo(camera);
	batch.setProjectionMatrix(camera.combined);
	batch.begin();
	for(Sprite sprite : worldController.testSprites) {
		sprite.draw(batch);
	}
	batch.end();
}
```

The `applyTo()` method should be called on each frame right at the beginning in the `renderTestObjects()` method of `WorldRenderer`. It will take care of correctly setting up the camera object that is passed. {{一帧只需要设置一次摄像头}}


下面是直接控制摄像头的键（`WorldController`）：
```java
	private void handleDebugInput (float deltaTime) {
		if (Gdx.app.getType() != ApplicationType.Desktop) return;
		// Selected Sprite Controls
		float sprMoveSpeed = 5 * deltaTime;
		if (Gdx.input.isKeyPressed(Keys.A))
			moveSelectedSprite(-sprMoveSpeed, 0);
		if (Gdx.input.isKeyPressed(Keys.D))
			moveSelectedSprite(sprMoveSpeed, 0);
		if (Gdx.input.isKeyPressed(Keys.W))
			moveSelectedSprite(0, sprMoveSpeed);
		if (Gdx.input.isKeyPressed(Keys.S))
			moveSelectedSprite(0, -sprMoveSpeed);
		// Camera Controls (move)
		float camMoveSpeed = 5 * deltaTime;
		float camMoveSpeedAccelerationFactor = 5;

		if (Gdx.input.isKeyPressed(Keys.SHIFT_LEFT))
			camMoveSpeed *= camMoveSpeedAccelerationFactor;
		if (Gdx.input.isKeyPressed(Keys.LEFT))
			moveCamera(-camMoveSpeed, 0);
		if (Gdx.input.isKeyPressed(Keys.RIGHT))
			moveCamera(camMoveSpeed, 0);
		if (Gdx.input.isKeyPressed(Keys.UP))
			moveCamera(0, camMoveSpeed);
		if (Gdx.input.isKeyPressed(Keys.DOWN))
			moveCamera(0, -camMoveSpeed);
		if (Gdx.input.isKeyPressed(Keys.BACKSPACE))
			cameraHelper.setPosition(0, 0);
		// Camera Controls (zoom)
		float camZoomSpeed = 1 * deltaTime;
		float camZoomSpeedAccelerationFactor = 5;
		if (Gdx.input.isKeyPressed(Keys.SHIFT_LEFT))
			camZoomSpeed *= camZoomSpeedAccelerationFactor;
		if (Gdx.input.isKeyPressed(Keys.COMMA))
			cameraHelper.addZoom(camZoomSpeed);
		if (Gdx.input.isKeyPressed(Keys.PERIOD))
			cameraHelper.addZoom(-camZoomSpeed);
		if (Gdx.input.isKeyPressed(Keys.SLASH))
			cameraHelper.setZoom(1);
		}
	private void moveCamera (float x, float y) {
		x += cameraHelper.getPosition().x;
		y += cameraHelper.getPosition().y;
		cameraHelper.setPosition(x, y);
	}
```

## 4 收集资源

### 4.1 设置 Android 应用图标

### 4.2 创建 texture atlases

A 纹理贴图(*texture atlas*) (also known as a *sprite sheet*) is just an ordinary image file that can be rendered to the screen like any other image. 那么它有何特殊之处？It is used as a container image that holds several smaller subimages arranged in such a way that they do not overlap each other and still fit into the size of the texture atlas. This way, we can greatly reduce the amount of textures that are sent to the graphics processor, which will significantly improve the overall render performance. The texture atlases are especially useful for games where a lot of small and different images are rendered at once. The reason for this is that switching between different textures is a very costly process. Each time you change textures while rendering, new data needs to be sent into video memory. If you use the same texture for everything, this can be avoided.

纹理贴图不仅能显著提高游戏的帧率，but will also allow us to use subimages as **Non-Power-Of-Two(NPOT)** textures. 但纹理贴图的长和宽应该总是2的整数倍，避免在不支持 NPOT 纹理的硬件上出现问题。子图象的大小可以任意的原因是，2的整数倍的规则只作用于加载到图像内存纹理。Therefore, when we actually render a subimage, we are still using the texture atlas, which is a power-of-two texture as our pixel source; however, we will only use a certain part of it as our final texture to draw something.

Libgdx 有一个内建的纹理打包器（packer），用于自动化创建和刷新纹理贴图的过程。下面是将所有游戏对象的图标放入一个贴图（atlas）的效果：

![](atlas_for_game.png)

The 
purple border around each image is a debugging feature of Libgdx's texture packer that can be toggled on and off. It can be used to visualize the true size of your subimages which otherwise can be difficult to see if the subimages use transparency. Also when padding is enabled, which is the default by using two pixels for each direction, you would barely see the difference without the enabled debugging lines.

> Padding your images inside a texture atlas helps you avoid an issue that is known as *texture bleeding* (also known as *pixel bleeding*) while texture filtering and/or mip-mapping is enabled.
The texture filter mode can be set to smooth pixels of a texture. This is basically done by looking for the pixel information which is next to the current pixel that is to be smoothened. The problem here is that if there is a pixel of a neighboring subimage, its pixels may also be taken into account, which results in the unwanted effect of pixels bleeding from one subimage into another.

For the texture packer, some preparations need to be done beforehand since it is a so-called extension to Libgdx that is not a part of the core functionality. Go to `C:\libgdx\` and extract `extensions/gdx-tools.jar` from the `libgdx-0.9.7.zip` file. Put the `gdx-tools.jar` file in the `CanyonBunny-desktop/libs` subfolder. Next, the extension has to be added to Build Pathin Eclipse.

We will now add the code to automate the generation process of the texture atlas. Create a new folder called `assets-raw` under *CanyonBunny-desktop*. Also, add a subfolder named `assets-raw/images`. This is where we put our image files to be included in the texture atlas.

Next, add the following two lines of code:
```java
import com.badlogic.gdx.tools.imagepacker.TexturePacker2;
import com.badlogic.gdx.tools.imagepacker.TexturePacker2.Settings;
```

Then, apply the following changes to `Main` in CanyonBunny-desktop:
```java
public class Main {
	private static boolean rebuildAtlas = true;
	private static boolean drawDebugOutline = true;
	public static void main (String[] args) {
		if (rebuildAtlas) {
			Settings settings = new Settings();
			settings.maxWidth = 1024;
			settings.maxHeight = 1024;
			settings.debug = drawDebugOutline;
			TexturePacker2.process(settings, "assets-raw/images",
				"../CanyonBunny-android/assets/images", 
				"canyonbunny.pack");
		}
		LwjglApplicationConfiguration cfg =
			new LwjglApplicationConfiguration();
		cfg.title = "CanyonBunny";
		cfg.useGL20 = false;
		cfg.width = 800;
		cfg.height = 480;
		new LwjglApplication(new CanyonBunnyMain(), cfg);
	}
}
```

`TexturePacker2.process()`第一个参数是`Settings`对象，可选。接下来两个参数是源文件夹和目标文件夹。最后一个参数是，name of the description file that is needed to load and use the texture atlas. The description file (in our example, `canyonbunny.pack`) will be created by `TexturePacker2` and will contain all the information about all the subimages, 例如它们在纹理贴图中的位置、大小和偏移。

The `maxWidth` and `maxHeight` variables of the `Settings` instance define the maximum dimensions (in pixels) for the texture atlas. 确保单个子图像的高度和宽度不要超过设置的最大大小。*Padding* the subimages in the atlas will reduce the available size a little bit more, so make sure to take this factor into account also. The `debug` variable controls whether the debug lines should be added to the atlas or not.

> If the texture packer cannot fit all subimages into a single texture, it will automatically split them up into several texture atlases. However, there is a chance that subimages are distributed in an unfavorable way between those atlases, which in turn could have an impact on render performance.
Libgdx's texture packer has a very smart feature to tackle this type of problem. All you need to do is group the subimages in their own subfolder in `assets-raw`. This way the texture packer will create one image file per subfolder that belongs to the texture atlas. You have to use the full path to the subimage if you want to use this functionality; for example, a subimage `assets-raw/items/gold_coin.png` would be referenced as `items/gold_coin`.

上面是通过代码创建纹理贴图的方式。但更直观的方式是使用*TexturePacker-GUI*。官网：https://code.google.com/p/libgdx-texturepacker-gui/。

![](TexturePackerGUI.png)

There is also a popular commercial tool called **TexturePacker** for creating texture atlases. This tool has been developed by Andreas Löw and is available for all three major platforms. Formore information, check out the official website at http://www.codeandweb.com/texturepacker.

### 4.3 加载和追踪 assets






