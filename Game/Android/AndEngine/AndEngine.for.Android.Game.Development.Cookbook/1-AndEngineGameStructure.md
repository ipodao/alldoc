# 1. AndEngine游戏结构

## 1.2 生命周期

参见`PacktRecipesActivity`类。

创建一个Activity作为我们游戏的入口。有一些AndEngine生命周期方法需要我们定义，包括创建`EngineOptions`对象，创建`Scene`对象，用子实体（entities）填充场景（Scene）。这些方法按以下顺序调用：

1、定义`onCreateEngineOptions()`方法：

	@Override
	public EngineOptions onCreateEngineOptions() {
		// 定义mCamera对象
		mCamera = new Camera(0, 0, WIDTH, HEIGHT);
		// 定义引擎选项
		EngineOptions engineOptions = new EngineOptions(true,
			ScreenOrientation.LANDSCAPE_FIXED, 
			new FillResolutionPolicy(),
			mCamera);
		// 让屏幕一直亮着。
		/ /当游戏不活动时屏幕可能会黑掉，禁用此行为
		engineOptions.setWakeLockOptions(WakeLockOptions.SCREEN_ON);
		// Return the engineOptions object, passing it to the engine
		return engineOptions;
	}

2、定义`onCreateResources()`方法。加载资源。资源包括textures、声音、字体等。

	@Override
	public void onCreateResources(OnCreateResourcesCallback pOnCreateResourcesCallback) {
		// 加载完所有资源后，通知pOnCreateResourcesCallback
		// onCreateResourcesFinished() 应被最后调用
		pOnCreateResourcesCallback.onCreateResourcesFinished();
	}

3、定义`onCreateScene()`方法。实例化`Scene`对象。

	@Override
	public void onCreateScene(OnCreateSceneCallback pOnCreateSceneCallback) {
		// 创建Scene对象
		mScene = new Scene();
		// 通知回调，我们已加载完资源
		// 传入mScene，会传给mEngine，显示在界面上
		pOnCreateSceneCallback.onCreateSceneFinished(mScene);
	}

对于复杂的情况，可能还需要安装触摸事件监听器，更新handlers等。

4、定义`onPopulateScene()`方法：

	@Override
	public void onPopulateScene(Scene pScene, OnPopulateSceneCallback pOnPopulateSceneCallback) {
		// 填充完Scene后再调用onPopulateSceneFinished()
		pOnPopulateSceneCallback.onPopulateSceneFinished();
	}

启动时调用的生命周期回调依次是：

* `onCreate`：Android应用标准入口。在AndEngine中，该方法仅是调用`onCreateEngineOptions()`。
* `onResume`：Android标准方法。Here, we simply acquire the wake lock settings from our EngineOptions object and proceed to call the onResume() method for the engine's RenderSurfaceView object.
* `onSurfaceCreated`：This method will either call `onCreateGame()` during the initial startup process of our activity or register a Boolean variable as true for resource reloading if the activity had previously been deployed.
* `onReloadResources`：This method reloads our game resources if our application is brought back into focus from minimization. 应用初始执行时不调用该方法。
* `onCreateGame`：This is in place to handle the order of execution of the next three callbacks in the AndEngine life cycle.
* `onCreateResources`：声明和定义启动Activity所需的初始资源，如textures、声音、字体等。
* `onCreateScene`：处理活动`Scene`对象的初始化。可以在该方法中，向`Scene`添加实体。但处于结构清晰的原因，最好在`onPopulateScene()`中添加实体。
* `onPopulateScene`：此时安装scene就要完成，还有几个生命周期回调会被引擎自动调用。此方法用于定义游戏首次启动时`Scene`上的可视内容。注意此时`Scene`已经被创建且与引擎关联。若没有启动屏，且实体数量多，可能观察到实体被添加到`Scene`的过程。
* `onGameCreated`：表示`onCreateGame()`序列已经结束，加载资源（如果需要）或什么也不做。根据`onSurfaceCreated`判断是否要重新加载资源。
* `onSurfaceChanged`：每次应用朝向改变后调用。
* `onResumeGame`：活动启动周期的最后一个方法。如果活动到达这步，引擎的`start()`会被调用，bringing the game's *update thread* to life。

最小化/终止调用的生命周期方法：

* `onPause`：第一个被调用的方法。调用`RenderSurfaceView`的pause方法，and reverts the wake lock settings applied by the game engine。
* `onPauseGame`：调用引擎的`stop()`方法，causing all of the Engine's update handlers to halt along with the update thread。
* `onDestroy`：In the onDestroy() method, AndEngine clears all graphical resources contained within `ArrayList` objects held by the Engine's manager classes. These managers include the `VertexBufferObjectManager` class, the `FontManager` class, the `ShaderProgramManager` class, and finally the `TextureManager` class.
* `onDestroyResources`：This method name may be a little misleading since we've already unloaded the majority of resources in onDestroy(). What this method really does is release all of the sound and music objects stored within the respective managers by calling their `releaseAll()` methods.
* `onGameDestroyed`：Finally, we reach the last method call required during a full AndEngine life cycle. Not a whole lot of action takes place in this method. AndEngine simply sets an `mGameCreated` Boolean variable used in the Engine to false, which states that the activity is no longer running.

![](liftcycle.png)

> 由于AndEngine生命周期异步的特性，it is possible for some methods to be executed multiple times during a single startup instance. The occurrence of these events varies between devices.

除了`BaseGameActivity`，还可以考虑使用以下类。

**LayoutGameActivity**

The `LayoutGameActivity` class is a useful activity class that allows us to incorporate the AndEngine scene-graph view into an ordinary Android application. 利用该类，可以在游戏中防止Android原生View（如按钮、布局等）。始终这种活动最常见的原因是，轻松实现游戏内广告。

使用`LayoutGameActivity`还需要：

1、向工程默认的布局XML文件添加以下代码。该文件一般名为`main.xml`。The following code snippet adds the AndEngine `RenderSurfaceView` class to our layout file. 该View用于在设备上显示游戏：

	<org.andengine.opengl.view.RenderSurfaceView
		android:id="@+id/gameSurfaceView"
		android:layout_width="fill_parent"
		android:layout_height="fill_parent"/>

2、 在活动中给出布局的id和`RenderSurfaceView`的id：

	@Override
	protected int getLayoutID() {
		return R.layout.main;
	}
	@Override
	protected int getRenderSurfaceViewID() {
		return R.id.gameSurfaceView;
	}

**SimpleBaseGameActivity和SimpleLayoutGameActivity**

`SimpleBaseGameActivity`和`SimpleLayoutGameActivity`简化了生命周期回调的实现。They do not require us to override the onPopulateScene() method and on top of that, we are not required to make calls to the method callbacks when we are finished defining the overridden methods. With these activity types, we can simply add the unimplemented life cycle methods and AndEngine will handle the callbacks for us.

**SimpleAsyncGameActivity**

该类包含另外三个生命周期回调方法，分别是`onCreateResourcesAsync()`, `onCreateSceneAsync()`, `onPopulateSceneAsync()` along with the usual `onCreateEngineOptions()` method. The main difference between this activity and others is that it provides us with loading bars for each of the "Async" methods. The following snippet shows how we can increment the loading bar in the event of a texture being loaded:

	@Override
	public void onCreateResourcesAsync(IProgressListener pProgressListener)
		throws Exception {
		// Load texture number one
		pProgressListener.onProgressChanged(10);
		// Load texture number two
		pProgressListener.onProgressChanged(20);
		// Load texture number three
		pProgressListener.onProgressChanged(30);
		// We can continue to set progress to whichever value we'd like
		// for each additional step through onCreateResourcesAsync...
	}

## 1.3 选择引擎类型

AndEngine包含几种引擎。不同的引擎游戏风格完全不同。

想选择引擎类型，覆盖`onCreateEngine()`方法。该方法允许我们返回一个定制的引擎对象。

	@Override
	public Engine onCreateEngine(EngineOptions pEngineOptions) {
		return super.onCreateEngine(pEngineOptions);
		// super方法做了以下事情：
		// return new Engine(pEngineOptions);
	}


下面简要介绍各种引擎。

### `Engine`

`Engine`不适合多数游戏。因为它完全不限制FPS（frames per second）。在不同的设备上，游戏速度可能不同。For this reason, noticeable issues can arise in devices which might not run as fast, especially when physics are a big part of the game. 若要使用这种引擎，不需要其他步骤了。

### `FixedStepEngine`

游戏开发时理想的引擎。它限制游戏循环按固定速度更新。例如，下面定义每秒60步：

	@Override
	public Engine onCreateEngine(EngineOptions pEngineOptions) {
		return new FixedStepEngine(pEngineOptions, 60);
	}

### `LimitedFPSEngine`

限制FPS。This will cause the Engine to do some internal calculations, and if the difference between the preferred FPS is greater than the current FPS that the Engine is achieving, the Engine will wait a fraction of a second before proceeding with the next update. 第二个参数指定最大FPS。

	@Override
	public Engine onCreateEngine(EngineOptions pEngineOptions) {
		return new LimitedFPSEngine(pEngineOptions, 60);
	}

### `SingleSceneSplitScreenEngine`和`DoubleSceneSplitScreenEngine`

`SingleSceneSplitScreenEngine`和`DoubleSceneSplitScreenEngine`允许我们创建带有两个独立摄像机的游戏，有一个常见（常用于单人游戏），或两个常见（多人游戏）。还有其他广泛的用途，如小地图，多个perspectives，菜单系统等。See Chapter 4, Creating a Split-screen Game, for more specific details on setting up these types of Engine objects.

## 1.4 选择分辨率（resolution）策略

Android设备分辨率差异很大。Generally developers and users alike prefer that a game takes up the full width and height of the device's display, but in some cases our resolution policy may need to be carefully selected in order to properly display our scenes as we—the developer—see fit. 我们将讨论多种可用的分辨率策略，分析哪种适合我们。

