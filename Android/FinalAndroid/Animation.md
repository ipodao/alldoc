# 动画

## 属性动画与View动画的区别

View动画只能动画View对象。且只能动画View的部分属性。

View动画的另一个缺点是，它只修改View绘制的位置，而不是View自身。例如，移动按钮后，实际可点击按钮的位置不变。

属性动画修改对象自身。

View动画编程时代码简单。如果View动画够用就要View动画。

## Drawable动画

最简单的动画是按顺序显示一组drawables。这被称为drawable动画。要创建此种动画，创建一个XML文件，列出参与动画的drawables。可以控制持续时间，及是否重复播放。

例子，动画的球：

1、创建一个简单的形状drawable。分别创建三个颜色的。下面是白色的：

	<?xml version="1.0" encoding="utf-8"?>
	<shape xmlns:android=”http://schemas.android.com/apk/res/android”
		android:shape=”oval” >
		<solid android:color=”#FFFFFF”/>
		<size android:height=”100dp” android:width=”100dp” />
	</shape>

2、创建drawable动画。令动画运行一次后停止（onshot）。时长250毫秒。

	<?xml version=”1.0” encoding=”utf-8”?>
	<animation-list xmlns:android=”http://schemas.android.com/apk/res/android”
		android:visible=”true” android:oneshot=”true”>
		<item android:drawable=”@drawable/white_circle” android:duration=”250” />
		<item android:drawable=”@drawable/gray_circle” android:duration=”250” />
		<item android:drawable=”@drawable/black_circle” android:duration=”250” />
	</animation-list>

android:visible表示在动画开始前，animated drawable可见。

3、创建一个布局，一个ImageView，其远是动画：

	<?xml version=”1.0” encoding=”utf-8”?>
	<ImageView xmlns:android=”http://schemas.android.com/apk/res/android”
		android:id=”@+id/image_view”
		android:layout_width=”match_parent”
		android:layout_height=”match_parent”
		android:scaleType=”center”
		android:src=”@drawable/animation” />
	
4、创建Activity。Because the animation is a one-shot, you need to call stop() before start() for subsequent taps: 

	public class AnimationExampleActivity extends Activity {
		@Override
		public void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.main);
			ImageView iv = (ImageView) findViewById(R.id.image_view);
			iv.setOnTouchListener(new OnTouchListener() {
				@Override
				public boolean onTouch(View v, MotionEvent event) {
					ImageView iv = (ImageView) v;
					AnimationDrawable ad = (AnimationDrawable) iv.getDrawable();
					ad.stop();
					ad.start();
					return true;
				}
			});
		}
	}
	
## View动画

View动画框架是Android 3.0之前的主要的动画框架。View animation provides a series of built-in animation operations that generate in-between, or tween, views. 你指定起始和终止值，动画框架改变显示的视图。该系统实现简单，但有一些缺点：

- 只能作用域View对象。不能作用于其他对象。
- 只能操作一组默认属性，不能影响其他属性。
- View动画只影响View如何被绘制，不影响其定位。Importantly, the rectangular hit area of views is not moved during a view animation—a common bug found in animation code. You need to alter the view after the animation is finished to update its position. 

这些限制导致Android 3.0后不推荐使用View动画。

### 定义动画

动画可以在XML或代码中定义。推荐使用XML，因为简单和可重用。View动画的XML放置在`res/anim/`。View动画有以下可用选项：

- translate：移动一个View
- scale：改变View大小
- rotate：旋转View
- alpha：改变View透明度

一个移动View的例子：

	<?xml version=”1.0” encoding=”utf-8”?>
	<translate xmlns:android=”http://schemas.android.com/apk/res/android”
		android:duration=”500”
		android:toYDelta=”25%”
		android:toXDelta=”25%” />

上面设置的是动画时长和终止位置。位置指定为View大小的百分比，该动画会将View移到右下。位置可以指定为View大小的百分比，父View大小的百分比，或像素值（无单位）。

等效于上面XML的Java代码：

	TranslateAnimation anim = new TranslateAnimation(
	TranslateAnimation.RELATIVE_TO_SELF, 0.0f, 
	TranslateAnimation.RELATIVE_TO_SELF, 0.25f, 
	TranslateAnimation.RELATIVE_TO_SELF, 0.0f, 
	TranslateAnimation.RELATIVE_TO_SELF, 0.25f);
	anim.setDuration(500);

多个动画可以放入一个Set，令它们同时发生。

	<?xml version=”1.0” encoding=”utf-8”?>
	<set xmlns:android=”http://schemas.android.com/apk/res/android”>
		<translate
			android:duration=”500”
			android:toYDelta=”25%”
			android:toXDelta=”25%” />
		<alpha
			android:duration=”500”
			android:fromAlpha=”1.0”
			android:toAlpha=”0.0” />
	</set>

组内动画不一定需要同时进行。可以为每个动画指定`android:startOffset`特性。该特性表示在整组动画开始后，延迟指定时间再指定特定动画。下面alpha动画会等待translate动画完成后开始：

	<?xml version=”1.0” encoding=”utf-8”?>
	<set xmlns:android=”http://schemas.android.com/apk/res/android”>
		<translate
			android:duration=”500”
			android:toYDelta=”25%”
			android:toXDelta=”25%” />
		<alpha
			android:duration=”500”
			android:startOffset=500”
			android:fromAlpha=”1.0”
			android:toAlpha=”0.0” />
	</set>

### 使用interpolator

动画默认是线性播放的。利用interpolator改变动画在时间上的进行方式。

支持的interpolator有accelerate, decelerate, overshoot, bounce等。

	<translate xmlns:android=”http://schemas.android.com/apk/res/android”
		android:duration=”500”
		android:toYDelta=”50%”
		android:toXDelta=”50%”
		android:interpolator=”@android:anim/accelerate_interpolator”/>

### 使用动画

对View施加动画，调用View的`startAnimation`方法，传入要运行的动画：

	TextView tv = (TextView) findViewById(R.id.text);
	Animation animation = AnimationUtils.loadAnimation(this, R.anim.slide);
	tv.startAnimation(animation);

上述动画令`TextView`移向右下。动画完成后，View将回到之前的位置。记住，动画只是影响View的绘制，而不会改变View对象。当动画完成后，View被重新绘制，显示在该出现的位置。为解决该问题，可以利用`AnimationListener`在动画完成后，改变View。例子，在动画完成后令View消失：

	final TextView tv = (TextView) findViewById(R.id.text1);
	Animation animation = AnimationUtils.loadAnimation(this, R.anim.slide);
	animation.setAnimationListener(new AnimationListener() {
		@Override
		public void onAnimationStart(Animation animation) {}
		@Override
		public void onAnimationRepeat(Animation animation) {}
		@Override
		public void onAnimationEnd(Animation animation) {
			tv.setVisibility(View.INVISIBLE);
		}
	});
	tv.startAnimation(animation);

提示：除了`AnimationListener`，还可以将`fillAfter`特性设为true。这将导致动画的结果被保持住。但注意，这只是影响View的显示。如果允许用户tap View，还是需要animation listener切实的移动它。

### 例子

参见Android UI Fundamental: addiNGa CloCk-FliPPiNG aNimatioNto the timetraCke

## 属性动画

Android 3.0引入属性动画。它可以实际改变任何对象。用它创建动画更简单。

可以动画任何对象的属性。不管它是否绘制在屏幕上。

可以定义动画的以下特征：

- 持续时间：默认300毫秒。
- 时间插值法（Time interpolation）：属性值是动画当前已过时间（elapsed time）的函数。
- 重复次数和行为：是否重复，重复几次。是否反向播放。Setting it to reverse plays the animation forwards then backwards repeatedly, until the number of repeats is reached.
- Animator sets: 可以将动画分组，同时播放，顺序播放，延迟播放。
- 帧刷新延迟：刷新动画帧的频率。默认周期是10毫秒，但最总的刷新频率取决于系统的繁忙程度，and how fast the system can service the underlying timer.

### 属性动画原理

兼得例子。一个对象，动画x属性（在屏幕上的水平位置）。持续时间设为40 ms，距离为40像素。每隔10ms（默认帧刷新频率），对象水平移动40像素。这里使用线性插值，匀速移动。

也可以非线性插值。

属性动画系统中的主要类：

![](valueanimator.png)

`ValueAnimator`控制动画时序（timing）, 如动画已运行事件，动画属性当前值。

`ValueAnimator`使用`TimeInterpolator`定义动画插值，`TypeEvaluator`定义如何计算动画的属性值。例如`TimeInterpolator`可以是`AccelerateDecelerateInterpolator`，`TypeEvaluator`可以是`IntEvaluator`。

`ValueAnimator`计算的`elapsed fraction`取值0到1。1表示完成。`ValueAnimator`计算出`elapsed fraction`，调用`TimeInterpolator`，根据插值法，计算`interpolated fraction`。计算出`interpolated fraction`后，`ValueAnimator`调用`TypeEvaluator`，计算动画属性的值。

API Demo中`com.example.android.apis.animation`包提供了很多例子。

### API概览

属性动画API参见`android.animation`。View动画已经定义了不少插值类（`android.view.animation`）。我们可以在属性动画总使用它们。

`Animator`类提供了创建动画的基本结构。一般不直接使用该类。而是用它的子类：

- `ValueAnimator` 动画一个属性有两个任务：1、决定属性值；2、将这个值设置到对象的属性上。`ValueAnimator`只完成第一步，因此你需要监听值的改变，自己修改对象。
- `ObjectAnimator` `ValueAnimator`的子类，允许设置目标对象和目标属性。该类会将新值更新到对象的动画属性上。多数情况下应该使用这个类。注意，`ObjectAnimator`对动画属性有一些要求。｛｛用反射的缘故｝｝
- `AnimatorSet` 可以将动画分组，同时播放，顺序播放，延迟播放。

取值器（Evaluators）如何计算特定类型属性值。有以下可能计算器：

- `IntEvaluator`： `int`属性的计算器。
- `FloatEvaluator`： `float`属性的计算器。
- `ArgbEvaluator`： 颜色的默认计算器。表示成十六进制值。
- `TypeEvaluator`： 接口，用于自定义计算器。如果属性值不是上述类型，序自定义计算器。

时间插值指定时间与属性值的关系。

- `AccelerateDecelerateInterpolator` 开始和结束慢，中间加速。
- `AccelerateInterpolator` 开始慢，然后加速。
- `AnticipateInterpolator` An interpolator whose change starts backward then flings forward.
- `AnticipateOvershootInterpolator` An interpolator whose change starts backward, flings forward and overshoots the target value, then finally goes back to the final value.
- `BounceInterpolator` An interpolator whose change bounces at the end.
- `CycleInterpolator` An interpolator whose animation repeats for a specified number of cycles.
- `DecelerateInterpolator` An interpolator whose rate of change starts out quickly and and then decelerates.
- `LinearInterpolator` An interpolator whose rate of change is constant.
- `OvershootInterpolator` An interpolator whose change flings forward and overshoots the last value then comes back.
- `TimeInterpolator` 接口，自定义实现。

### ValueAnimator

可以通过`ValueAnimator`的工厂方法获取一个`ValueAnimator`：ofInt(), ofFloat(), ofObject()。

	ValueAnimator animation = ValueAnimator.ofFloat(0f, 1f);
	animation.setDuration(1000);
	animation.start();

其他类型的属性的动画：

	ValueAnimator animation = ValueAnimator.ofObject(new MyTypeEvaluator(),
		startPropertyValue, endPropertyValue);
	animation.setDuration(1000);
	animation.start();

上面所有代码实际没有效果，因为`ValueAnimator`不直接操纵对象属性。如果想把对象属性修改成计算后的值，需要定义监听器`ValueAnimator.AnimatorUpdateListener`。

### ObjectAnimator

`ObjectAnimator`能直接修改对象属性。

	ObjectAnimator anim = ObjectAnimator.ofFloat(foo, "alpha", 0f, 1f);
	anim.setDuration(1000);
	anim.start();

要想都规划的对象属性必须有setter方法，`set<propertyName>()`。

如果`values...`参数，只指定了一个值，这个值被当作结束值。于是对象属性必须有getter方法，`get<propertyName>()`，用于获取起始值。

getter方法、setter方法和起始、最终值的类型要一致。

有时你需要自己调用View的`invalidate()`，引发其重绘。可以在`onAnimationUpdate()`中做。For example, animating the color property of a Drawable object only cause updates to the screen when that object redraws itself. View的属性setter，如`setAlpha()`和`setTranslationX()`自己会invalidate，因此对于这些方法不用调用invalidate。

### 使用AnimatorSet编组多个动画

AnimatorSet对象本身也可以嵌套。

The following sample code taken from the Bouncing Balls sample (modified for simplicity) plays the following Animator objects in the following manner:

1. Plays bounceAnim.
1. Plays squashAnim1, squashAnim2, stretchAnim1, and stretchAnim2 at the same time.
1. Plays bounceBackAnim.
1. Plays fadeAnim.

代码：

	AnimatorSet bouncer = new AnimatorSet();
	bouncer.play(bounceAnim).before(squashAnim1);
	bouncer.play(squashAnim1).with(squashAnim2);
	bouncer.play(squashAnim1).with(stretchAnim1);
	bouncer.play(squashAnim1).with(stretchAnim2);
	bouncer.play(bounceBackAnim).after(stretchAnim2);
	ValueAnimator fadeAnim = ObjectAnimator.ofFloat(newBall, "alpha", 1f, 0f);
	fadeAnim.setDuration(250);
	AnimatorSet animatorSet = new AnimatorSet();
	animatorSet.play(bouncer).before(fadeAnim);
	animatorSet.start();

### 动画监听器

可以通过以下监听器监听动画工程中的重要事件：

**Animator.AnimatorListener**

- onAnimationStart() 动画开始
- onAnimationEnd() 动画结束
- onAnimationRepeat() 动画自己重复
- onAnimationCancel() 动画被取消。A cancelled animation also calls onAnimationEnd(), regardless of how they were ended.

`AnimatorListenerAdapte`r是`AnimatorListener`的适配器。

	ValueAnimatorAnimator fadeAnim = ObjectAnimator.ofFloat(newBall,
		"alpha", 1f, 0f);
	fadeAnim.setDuration(250);
	fadeAnim.addListener(new AnimatorListenerAdapter() {
		public void onAnimationEnd(Animator animation) {
		    balls.remove(((ObjectAnimator)animation).getTarget());
		}

**ValueAnimator.AnimatorUpdateListener**

- `onAnimationUpdate()` 每一帧动画调用一次。监听该事件，以获取`ValueAnimator`产生的值。To use the value, query the ValueAnimator object passed into the event to get the current animated value with the getAnimatedValue() method. 如果直接使用`ValueAnimator`，肯定要实现该接口。

### ViewGroups布局改变动画

The property animation system provides the capability to animate changes to `ViewGroup` objects as well as provide an easy way to animate View objects themselves.

动画`ViewGroup`内布局改变还可以通过`LayoutTransition`类。ViewGroup中的View被添加或删除、或改变可见性（`setVisibility()`）后，可以施加显示和隐藏动画。ViewGroup中的剩余View可以动画到它们的新位置。You can define the following animations in a `LayoutTransition` object by calling `setAnimator()` and passing in an Animator object with one of the following `LayoutTransition` constants:

- `APPEARING`：动画作用在要出现的内容上。
- `CHANGE_APPEARING`：由于容器增加项目，导致其他项目发生改变。动画作用在发生改变的项目上。
- `DISAPPEARING` 动画作用在被移除的内容上。
- `CHANGE_DISAPPEARING`：由于项目要被移除，导致其他项目发生改变。动画作用在发生改变的项目上。

可以为这四类事件定义自定义动画，or just tell the animation system to use the default animations。

The LayoutAnimations sample in API Demos shows you how to define animations for layout transitions and then set the animations on the View objects that you want to animate.

The `LayoutAnimationsByDefault` and its corresponding `layout_animations_by_default.xml` layout resource file show you how to enable the *default* layout transitions for ViewGroups in XML. 只需要把容器的`android:animateLayoutchanges`特性设为`true`。例如：

	<LinearLayout
	    android:orientation="vertical"
	    android:layout_width="wrap_content"
	    android:layout_height="match_parent"
	    android:id="@+id/verticalContainer"
	    android:animateLayoutChanges="true" />

Setting this attribute to `true` automatically animates Views that are added or removed from the ViewGroup as well as the remaining Views in the ViewGroup.

### TypeEvaluator

`TypeEvaluator`接口只有一个方法：`evaluate()`。下面是`FloatEvaluator`类的代码：

	public class FloatEvaluator implements TypeEvaluator {
	
	    public Object evaluate(float fraction, Object startValue, Object endValue) {
	        float startFloat = ((Number) startValue).floatValue();
	        return startFloat + fraction * (((Number) endValue).floatValue() - startFloat);
	    }
	}

> `TypeEvaluator`的`fraction`参数收到的是`interpolated fraction`，这是`TimeInterpolator`计算后的结果。

### Interpolators

As an example, how the default interpolator `AccelerateDecelerateInterpolator` and the LinearInterpolator calculate interpolated fractions are compared below. The `LinearInterpolator` has no effect on the elapsed fraction. The `AccelerateDecelerateInterpolator` accelerates into the animation and decelerates out of it. The following methods define the logic for these interpolators:

**AccelerateDecelerateInterpolator**

	public float getInterpolation(float input) {
	    return (float)(Math.cos((input + 1) * Math.PI) / 2.0f) + 0.5f;
	}

**LinearInterpolator**

	public float getInterpolation(float input) {
	    return input;
	}

### 指定关键帧（Keyframes）

A [Keyframe](http://developer.android.com/reference/android/animation/Keyframe.html) object consists of a time/value pair that lets you define a specific state at a specific time of an animation. Each keyframe can also have its own interpolator to control the behavior of the animation in the interval between the previous keyframe's time and the time of this keyframe.

To instantiate a Keyframe object, you must use one of the factory methods, ofInt(), ofFloat(), or ofObject() to obtain the appropriate type of `Keyframe`. You then call the `ofKeyframe()` factory method to obtain a `PropertyValuesHolder` object. Once you have the object, you can obtain an animator by passing in the `PropertyValuesHolder` object and the object to animate. The following code snippet demonstrates how to do this:

	Keyframe kf0 = Keyframe.ofFloat(0f, 0f);
	Keyframe kf1 = Keyframe.ofFloat(.5f, 360f);
	Keyframe kf2 = Keyframe.ofFloat(1f, 0f);
	PropertyValuesHolder pvhRotation = PropertyValuesHolder.ofKeyframe("rotation", kf0, kf1, kf2);
	ObjectAnimator rotationAnim = ObjectAnimator.ofPropertyValuesHolder(target, pvhRotation)
	rotationAnim.setDuration(5000ms);

For a more complete example on how to use keyframes, see the MultiPropertyAnimation sample in APIDemos.

### 动画View

Android 3.0，View增加了新属性，并增加了相应的getter和setter方法。

属性动画系统可以改变View的实际属性。此外属性改变后，Views还会自动调用`invalidate()`方法。View动画可以利用的属性有：

- `translationX` 和 `translationY`: 控制View相对于其左上角（由其容器设置）的偏移。
- `rotation`, `rotationX`, `rotationY`：2D旋转（`rotation`属性）和3D旋转。
- `scaleX` 和 `scaleY`：2D缩放（around its pivot point）。
- `pivotX` 和 `pivotY`：控制pivot点的位置。默认pivot点位于对象中心。
- `x` 和 `y`：These are simple utility properties to describe the final location of the View in its container, as a sum of the left and top values and translationX and translationY values.
- `alpha`：View的透明度。1不透明（默认），0完全透明。

To animate a property of a View object, such as its color or rotation value, all you need to do is create a property animator and specify the View property that you want to animate. For example:

	ObjectAnimator.ofFloat(myView, "rotation", 0f, 360f);

### ViewPropertyAnimator

`ViewPropertyAnimator`可以同时动画View的多个属性。它与`ObjectAnimator`类似，因为它修改View属性的实际值，但同时动画多个属性时更有效率。`ViewPropertyAnimator`的代码更精确、已读。下面展示它们的区别：

多个`ObjectAnimator`对象

	ObjectAnimator animX = ObjectAnimator.ofFloat(myView, "x", 50f);
	ObjectAnimator animY = ObjectAnimator.ofFloat(myView, "y", 100f);
	AnimatorSet animSetXY = new AnimatorSet();
	animSetXY.playTogether(animX, animY);
	animSetXY.start();

一个`ObjectAnimator`

	PropertyValuesHolder pvhX = PropertyValuesHolder.ofFloat("x", 50f);
	PropertyValuesHolder pvhY = PropertyValuesHolder.ofFloat("y", 100f);
	ObjectAnimator.ofPropertyValuesHolder(myView, pvhX, pvyY).start();

`ViewPropertyAnimator`

	myView.animate().x(50f).y(100f);

For more detailed information about ViewPropertyAnimator, see the corresponding Android Developers [blog post](http://android-developers.blogspot.com/2011/05/introducing-viewpropertyanimator.html).

### 利用XML声明属性动画

XML，可以在多个活动中重用动画。编辑动画序列更简单。

属性动画XML文件放入`res/animator/`，之前的View动画放入`res/anim/`。`animator`可以是其他名字，但*Eclipse ADT plugin (ADT 11.0.0+)*等工具期望它是这个名字。

支持以下标签：

- ValueAnimator `<animator>`
- ObjectAnimator `<objectAnimator>`
- AnimatorSet `<set>`

示例：

	<set android:ordering="sequentially">
	    <set>
	        <objectAnimator
	            android:propertyName="x"
	            android:duration="500"
	            android:valueTo="400"
	            android:valueType="intType"/>
	        <objectAnimator
	            android:propertyName="y"
	            android:duration="500"
	            android:valueTo="300"
	            android:valueType="intType"/>
	    </set>
	    <objectAnimator
	        android:propertyName="alpha"
	        android:duration="500"
	        android:valueTo="1f"/>
	</set>

In order to run this animation, you must *inflate* the XML resources in your code to an `AnimatorSet` object, and then set the target objects for all of the animations before starting the animation set. Calling `setTarget()` sets a single target object for all children of the `AnimatorSet` as a convenience. The following code shows how to do this:

	AnimatorSet set = (AnimatorSet) AnimatorInflater.loadAnimator(myContext,
	    R.anim.property_animator);
	set.setTarget(myObject);
	set.start()

For information about the XML syntax for defining property animations, see [Animation Resources](http://developer.android.com/guide/topics/resources/animation-resource.html#Property).