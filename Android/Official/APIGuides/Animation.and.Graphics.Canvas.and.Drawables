## Canvas and Drawables

Android提供一组2D绘图API，可以在Canvas是那个绘自己的图形，或修改存在在的View，定制它们的外观。两种绘制2D图形的方式：
- Draw your graphics or animations into a View object from your layout. In this manner, the drawing of your graphics is handled by the system's normal View hierarchy drawing process — 你只要定义View中的图形。
- 直接在Canvas上绘图。此时，需要你去调相应类的`onDraw()`（`View.onDraw`）方法。(passing it your Canvas), 或Canvas的某个`draw...()`方法。这种方式还可以让你自己控制各种动画。

如果不需要图形动态改变，或不是在性能敏感的游戏中，应选择第一种，绘制到View上。例如，可以在View上显示静态图形或预定义动画。

如果应用需要定期重绘自己，则适合用第二个选择，绘制到Canvas。例如游戏。有两种重绘方式：
- 在Activity所在的线程，对创建的自定义View，调用`invalidate() `，然后处理`onDraw()`回调。
- 在另一个线程，即管理`SurfaceView`的线程，以这个线程最大可能的速度向Canvas绘制，不需要调用`invalidate()`。

### Draw with a Canvas

A Canvas works for you as a pretense, or interface, to the actual surface upon which your graphics will be drawn — it holds all of your "draw" calls. 利用Canvas，你实际是在一个底层的`Bitmap`上绘制，which is placed into the window.

`onDraw()`方法会想你提供一个Canvas实例，你调用它的方法。如果使用的是`SurfaceView`，可以通过`SurfaceHolder.lockCanvas()`方法获取一个Canvas。

如果你要定义一个新的Canvas，则你必须定义一个Bitmap，绘制发生在其上。**Canvas总是需要一个Bitmap**。创建一个新的Canvas：

```java
Bitmap b = Bitmap.createBitmap(100, 100, Bitmap.Config.ARGB_8888);
Canvas c = new Canvas(b);
```

利用上面新建的Canvas完成绘制后，你需要将底层`Bitmap`传给其他Canvas（通过`Canvas.drawBitmap(Bitmap,...)`）。因为，推荐最终要使用`View.onDraw()`或`SurfaceHolder.lockCanvas()`提供的Canvas。｛｛图形才能出现的屏幕上｝｝

在Canvas上绘制可以用上两类方法。Canvas类有一组绘制方法，如`drawBitmap(...)`、`drawRect(...)`、`drawText(...)`。其他可能用到的类也有一些`draw()`方法。如，若想向Canvas添加一个`Drawable`，可以调用`Drawable.draw()`方法，传入`Canvas`对象。

#### On a View

如果你的应用对帧率没有高要求，应该考虑自定义一个View，在`View.onDraw()`方法中利用Canvas绘制。The most convenient aspect of doing so is that the Android framework will provide you with a pre-defined Canvas to which you will place your drawing calls.

首先，继承`View`类（或子类），定义`onDraw()`方法。该方法由Android框架调用。只会在必要时调用。Each time that your application is prepared to be drawn, you must request your View be invalidated by calling `invalidate()`. This indicates that you'd like your View to be drawn and Android will then call your onDraw() method.

> 在非UI线程中请求invalidate，调用`postInvalidate()`。

参见[Building Custom Components](http://developer.android.com/guide/topics/ui/custom-components.html)。

For a sample application, see the Snake game, in the SDK samples folder: `/samples/Snake/`.

#### On a SurfaceView

`SurfaceView`是`View`的一个子类，offers a dedicated drawing surface within the View hierarchy. The aim is to offer this drawing surface to an application's secondary thread, so that the application isn't required to wait until the system's View hierarchy is ready to draw. Instead, a secondary thread that has reference to a SurfaceView can draw to its own Canvas at its own pace.

首先创建一个`SurfaceView`的子类。子类还需要实现`SurfaceHolder.Callback`。This subclass is an interface that will notify you with information about the underlying Surface, such as when it is created, changed, or destroyed. These events are important so that you know when you can start drawing, whether you need to make adjustments based on new surface properties, and when to stop drawing and potentially kill some tasks. Inside your `SurfaceView` class is also a good place to define your secondary `Thread` class, which will perform all the drawing procedures to your Canvas.

Instead of handling the Surface object directly, you should handle it via a `SurfaceHolder`. So, when your `SurfaceView` is initialized, get the `SurfaceHolder` by calling `getHolder()`. You should then notify the `SurfaceHolder` that you'd like to receive `SurfaceHolder` callbacks (from `SurfaceHolder.Callback`) by calling `addCallback()` (pass it `this`). Then override each of the `SurfaceHolder.Callback` methods inside your `SurfaceView` class.

In order to draw to the Surface Canvas from within your second thread, you must pass the thread your `SurfaceHandler` and retrieve the Canvas with `lockCanvas()`. You can now take the Canvas given to you by the `SurfaceHolder` and do your necessary drawing upon it. Once you're done drawing with the Canvas, call `unlockCanvasAndPost()`, passing it your Canvas object. The Surface will now draw the Canvas as you left it. Perform this sequence of locking and unlocking the canvas each time you want to redraw.

> Note: On each pass you retrieve the Canvas from the SurfaceHolder, the previous state of the Canvas will be retained. In order to properly animate your graphics, you must re-paint the entire surface. For example, you can clear the previous state of the Canvas by filling in a color with `drawColor()` or setting a background image with `drawBitmap()`. Otherwise, you will see traces of the drawings you previously performed.

For a sample application, see the Lunar Lander game, in the SDK samples folder: `/samples/LunarLander/`.

### Drawables



