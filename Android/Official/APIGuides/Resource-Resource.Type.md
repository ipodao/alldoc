## （未）资源类型

### Drawable资源

一个Drawable资源是一个可以绘制在屏幕上的图形。可以通过[getDrawable(int)](http://developer.android.com/reference/android/content/res/Resources.html#getDrawable(int))获取，或通过`android:drawable`和`android:icon`特性指定。有以下几种类型：

- **Bitmap文件** 一个bitmap图形文件(.png, .jpg, or .gif)。创建一个[BitmapDrawable](http://developer.android.com/reference/android/graphics/drawable/BitmapDrawable.html)。
- **Nine-Patch文件** 一个PNG图片，指定可拉伸区域，允许图片根据内容天正调整大小｛｛图片作为文字背景时，可以根据文字大小调整背景大小｝｝(.9.png). 创建一个`NinePatchDrawable`。
- **层列表** 一个Drawables数组。在数组后面的元素绘制在上面。创建一个`LayerDrawable`。
- **状态列表** 不同的状态引用不同的bitmap（例如，当按钮被按下后使用不同的图）。创建一个`StateListDrawable`。
- **Level List** 一个XML文件，定义一个drawable，管理多个可替代的Drawables，每个分配一个最大值。Creates a `LevelListDrawable`.
- **Transition Drawable** 两个drawable资源淡入淡出（cross-fade）。创建一个`TransitionDrawable`。
- **Inset Drawable** An XML file that defines a drawable that insets another drawable by a specified distance. This is useful when a View needs a background drawble that is smaller than the View's actual bounds.
- **Clip Drawable** An XML file that defines a drawable that clips another Drawable based on this Drawable's current *level* value. Creates a `ClipDrawable`.
- **Scale Drawable** An XML file that defines a drawable that changes the size of another Drawable based on its current *level* value. Creates a `ScaleDrawable`
- **Shape Drawable** 一个XML文件，定义一个几何形状，包括颜色和渐变。Creates a `ShapeDrawable`.

> 一个颜色资源也可以被当作一个drawable。For example, when creating a state list drawable, you can reference a color resource for the `android:drawable` attribute (`android:drawable="@color/green"`).

#### Bitmap

Android支持三种格式：.png (preferred), .jpg (acceptable), .gif (discouraged)。

> 在构建阶段，aapt工具（无损压缩）会自动优化Bitmap文件。例如一个支持颜色不超过256色的真彩色（true-color）PNG会被转换成带色盘（color palette）的8-bit PNG。这样图片内存减少但指令不变。So be aware that the image binaries placed in this directory can change during the build. If you plan on reading an image as a bit stream in order to convert it to a bitmap, put your images in the `res/raw/` folder instead, where they will not be optimized.

##### Bitmap文件

Android为`res/drawable/`下的每个文件创建一个Drawable资源。

例如，文件位置`res/drawable/filename.png`。文件名做为资源名。编译后的资源类型是`BitmapDrawable`。引用方式`R.drawable.filename`、`@[package:]drawable/filename`。

例子：`res/drawable/myimage.png`

	<ImageView
	    android:layout_height="wrap_content"
	    android:layout_width="wrap_content"
	    android:src="@drawable/myimage" />

	Resources res = getResources();
	Drawable drawable = res.getDrawable(R.drawable.myimage);

参见：

- [2D Graphics](http://developer.android.com/guide/topics/graphics/2d-graphics.html)
- [BitmapDrawable](http://developer.android.com/reference/android/graphics/drawable/BitmapDrawable.html)

##### XML Bitmap

XML bitmap定义在XML中，指向一个bitmap文件。可以作为一个原始bitmap文件的别名。还可以指定bitmap的附加属性，如dithering和tiling。

> `<bitmap>`可以作为`<item>`的子元素。例如创建 *state list* 时，可以不用`<item>`的`android:drawable`特性，而是嵌入一个`<bitmap>`。

例如，文件位置`res/drawable/filename.xml`，文件名作为资源名。编译后的资源类型是`BitmapDrawable`。引用：`R.drawable.filename`、`@[package:]drawable/filename`。

语法：

	<?xml version="1.0" encoding="utf-8"?>
	<bitmap
	    xmlns:android="http://schemas.android.com/apk/res/android"
	    android:src="@[package:]drawable/drawable_resource"
	    android:antialias=["true" | "false"]
	    android:dither=["true" | "false"]
	    android:filter=["true" | "false"]
	    android:gravity=["top" | "bottom" | "left" | "right" | "center_vertical" |
	                      "fill_vertical" | "center_horizontal" | "fill_horizontal" |
	                      "center" | "fill" | "clip_vertical" | "clip_horizontal"]
	    android:mipMap=["true" | "false"]
	    android:tileMode=["disabled" | "clamp" | "repeat" | "mirror"] />

`<bitmap>`特性：

- `android:src` 必需。引用另一个drawable资源。
- `android:antialias` 布尔。antialiasing.
- `android:dither` 布尔。Enables or disables dithering of the bitmap if the bitmap does not have the same pixel configuration as the screen (for instance: a ARGB 8888 bitmap with an RGB 565 screen).
- `android:filter` 布尔。Enables or disables bitmap filtering. Filtering is used when the bitmap is shrunk or stretched to smooth its apperance.
- `android:gravity` The gravity indicates where to position the drawable in its container if the bitmap is smaller than the container. 可以用'|'分隔多个常量值：
	- `top` 放在容器顶部。不改变大小。
	- `bottom`
	- `left`
	- `right`
	- `center_vertical` 不改变大小。
	- `fill_vertical` 如果需要，增加对象的高度。填满容器。
	- `center_horizontal` 不改变大小。
	- `fill_horizontal` 如果需要，增加对象的水平长度。填满容器。
	- `center`水平和垂直居中，不改变大小。
	- `fill` **默认值**。如果需要，增加对象的水平和垂直大小。填满容器。
	- `clip_vertical` Additional option that can be set to have the top and/or bottom edges of the child clipped to its container's bounds. The clip is based on the vertical gravity: a top gravity clips the bottom edge, a bottom gravity clips the top edge, and neither clips both edges.
	- `clip_horizontal` Additional option that can be set to have the left and/or right edges of the child clipped to its container's bounds. The clip is based on the horizontal gravity: a left gravity clips the right edge, a right gravity clips the left edge, and neither clips both edges.
- `android:mipMap` 布尔。Enables or disables the mipmap hint. See `setHasMipMap()` for more information. 默认false。
- `android:tileMode` Defines the tile mode. When the tile mode is enabled, the bitmap is repeated. 启用tile模式后Gravity会被忽略。取值：
	- `disabled` **默认值**。不要铺满。
	- `clamp` Replicates the *edge color* if the shader draws outside of its original bounds
	- `repeat` Repeats the shader's image horizontally and vertically.
	- `mirror` Repeats the shader's image horizontally and vertically, alternating mirror images so that adjacent images always seam.

例子：

	<?xml version="1.0" encoding="utf-8"?>
	<bitmap xmlns:android="http://schemas.android.com/apk/res/android"
	    android:src="@drawable/icon"
	    android:tileMode="repeat" />
	
SEE ALSO:

- [Creating alias resources](http://developer.android.com/guide/topics/resources/providing-resources.html#AliasResources)

#### Nine-Patch

[NinePatch](http://developer.android.com/reference/android/graphics/NinePatch.html)是一个PNG图像，你可以定义可拉伸区域，当View的内容变大超过图像大小时，Android可以缩放。这种类型图片一般作为View背景，且至少一个维度是"wrap_content"，当View内容变大时，Nine-Patch图像随着放大，匹配View大小。Nine-Patch的例子是，Android标准按钮使用的背景。

Same as with a normal bitmap, you can reference a Nine-Patch file directly or from a resource defined by XML.

For a complete discussion about how to create a Nine-Patch file with stretchable regions, see the [2D Graphics document](http://developer.android.com/guide/topics/graphics/2d-graphics.html#nine-patch).

例子，`res/drawable/filename.9.png`文件，文件名做资源名，编译后的资源类型是`NinePatchDrawable`。引用资源`R.drawable.filename`、`@[package:]drawable/filename`。

例如，`res/drawable/myninepatch.9.png`，使用在按钮上：

	<Button
	    android:layout_height="wrap_content"
	    android:layout_width="wrap_content"
	    android:background="@drawable/myninepatch" />

参见：

- [2D Graphics](http://developer.android.com/guide/topics/graphics/2d-graphics.html#nine-patch)
- [NinePatchDrawable](http://developer.android.com/reference/android/graphics/drawable/NinePatchDrawable.html)

##### XML Nine-Patch

An XML Nine-Patch is a resource defined in XML that points to a Nine-Patch file. The XML can specify dithering for the image.

FILE LOCATION:
res/drawable/filename.xml
The filename is used as the resource ID.
COMPILED RESOURCE DATATYPE:
Resource pointer to a NinePatchDrawable.
RESOURCE REFERENCE:
In Java: `R.drawable.filename`
In XML: `@[package:]drawable/filename`

语法：

	<?xml version="1.0" encoding="utf-8"?>
	<nine-patch
	    xmlns:android="http://schemas.android.com/apk/res/android"
	    android:src="@[package:]drawable/drawable_resource"
	    android:dither=["true" | "false"] />

特性：

- `android:src` Drawable resource. Required. Reference to a Nine-Patch file.
- `android:dither` Boolean. Enables or disables dithering of the bitmap if the bitmap does not have the same pixel configuration as the screen (for instance: a ARGB 8888 bitmap with an RGB 565 screen).

例子：

	<?xml version="1.0" encoding="utf-8"?>
	<nine-patch xmlns:android="http://schemas.android.com/apk/res/android"
	    android:src="@drawable/myninepatch"
	    android:dither="false" />

#### Layer List

LayerDrawable管理一个drawables数组。数组中后面的元素绘制在最上面。

FILE LOCATION: `res/drawable/filename.xml`。文件名是资源名。编译后的资源类型`LayerDrawable`。引用`R.drawable.filename`、`@[package:]drawable/filename`。语法：

	<?xml version="1.0" encoding="utf-8"?>
	<layer-list
	    xmlns:android="http://schemas.android.com/apk/res/android" >
	    <item
	        android:drawable="@[package:]drawable/drawable_resource"
	        android:id="@[+][package:]id/resource_name"
	        android:top="dimension"
	        android:right="dimension"
	        android:bottom="dimension"
	        android:left="dimension" />
	</layer-list>

`<item>`中可以放`<bitmap>`。`<item>`的特性：

- `android:top` 整数。顶部偏移，单位像素。
- `android:right`
- `android:bottom`
- `android:left`

默认，所有items都会缩放适应容器View的大小。如果将图片放入不同位置，可能增加View大小，图片会缩放。要避免缩放，在`<item>`中嵌套`<bitmap>`，定义gravity为不缩放的值，如"center"。For example, the following <item> defines an item that scales to fit its container View:

	<item android:drawable="@drawable/image" />

为避免缩放，利用`<bitmap>`子元素：

	<item>
	  <bitmap android:src="@drawable/image"
	          android:gravity="center" />
	</item>

例子：` res/drawable/layers.xml`	

	<?xml version="1.0" encoding="utf-8"?>
	<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
	    <item>
	      <bitmap android:src="@drawable/android_red"
	        android:gravity="center" />
	    </item>
	    <item android:top="10dp" android:left="10dp">
	      <bitmap android:src="@drawable/android_green"
	        android:gravity="center" />
	    </item>
	    <item android:top="20dp" android:left="20dp">
	      <bitmap android:src="@drawable/android_blue"
	        android:gravity="center" />
	    </item>
	</layer-list>

This layout XML applies the drawable to a View:

	<ImageView
	    android:layout_height="wrap_content"
	    android:layout_width="wrap_content"
	    android:src="@drawable/layers" />

效果：

![](drawable_layers.png)

#### State List

`StateListDrawable`定义在XML中，使用几个不同的图像表示相同图形对的不同状态。例如，按钮的状态有被按下、有焦点等。

每个状态用一个`<item>`元素表示。

注意，匹配状态时不是按最优匹配，而是从上到下匹配，匹配到适合当前状态后停止。

FILE LOCATION: `res/drawable/filename.xml`。文件名就是资源名。编译后的资源类型是`StateListDrawable`。引用：`R.drawable.filename`、`@[package:]drawable/filename`。

SYNTAX:

	<?xml version="1.0" encoding="utf-8"?>
	<selector xmlns:android="http://schemas.android.com/apk/res/android"
	    android:constantSize=["true" | "false"]
	    android:dither=["true" | "false"]
	    android:variablePadding=["true" | "false"] >
	    <item
	        android:drawable="@[package:]drawable/drawable_resource"
	        android:state_pressed=["true" | "false"]
	        android:state_focused=["true" | "false"]
	        android:state_hovered=["true" | "false"]
	        android:state_selected=["true" | "false"]
	        android:state_checkable=["true" | "false"]
	        android:state_checked=["true" | "false"]
	        android:state_enabled=["true" | "false"]
	        android:state_activated=["true" | "false"]
	        android:state_window_focused=["true" | "false"] />
	</selector>

`<selector>`元素是根元素。它的特性有：

- `android:constantSize`：布尔。"true" if the drawable's reported internal size remains constant as the state changes (the size is the maximum of all of the states); "false" if the size varies based on the current state. 默认是false。
- `android:dither`：布尔。"true" to enable dithering of the bitmap if the bitmap does not have the same pixel configuration as the screen (for instance, an ARGB 8888 bitmap with an RGB 565 screen); "false" to disable dithering. Default is true.
- `android:variablePadding`：布尔。"true" if the drawable's padding should change based on the current state that is selected; "false" if the padding should stay the same (based on the maximum padding of all the states). Enabling this feature requires that you deal with performing layout when the state changes, 多数情况下是不支持的。默认false。

`<item>`的特性：

- `android:drawable`：必需。
- `android:state_pressed`：布尔。"true"表示当对象被按下时应该使用此项；"false" if this item should be used in the default, non-pressed state.
- `android:state_focused`：布尔。"true"表示对象拥有焦点时应该使用此项；"false" if this item should be used in the default, non-focused state.
- `android:state_hovered`：布尔。"true"表示对象hovered by a cursor是该使用此项; "false" if this item should be used in the default, non-hovered state. 一般该状态与"focused"状态的效果相同。Introduced in API level 14.
- `android:state_selected`:布尔。"true" if this item should be used when the object is the current user selection when navigating with a directional control (such as when navigating through a list with a d-pad); "false" if this item should be used when the object is not selected. The selected state is used when focus (android:state_focused) is not sufficient （例如焦点在list view上，但其中的项可以d-pad选择）。
- `android:state_checkable`：布尔。"true" if this item should be used when the object is checkable; "false" if this item should be used when the object is not checkable. (Only useful if the object can transition between a checkable and non-checkable widget.)
- `android:state_checked`：布尔。"true" if this item should be used when the object is checked; "false" if it should be used when the object is un-checked.
- `android:state_enabled`：布尔。"true" if this item should be used when the object is enabled (capable of receiving touch/click events); "false" if it should be used when the object is disabled.
- `android:state_activated`：Boolean. "true" if this item should be used when the object is activated as the persistent selection (such as to "highlight" the previously selected list item in a persistent navigation view); "false" if it should be used when the object is not activated. Introduced in API level 11.
- `android:state_window_focused`：Boolean. "true" if this item should be used when the application window has focus (the application is in the foreground), "false" if this item should be used when the application window does not have focus (for example, if the notification shade is pulled down or a dialog appears).

> Note: Remember that Android applies the first item in the state list that matches the current state of the object. So, if the first item in the list contains none of the state attributes above, then it is applied every time, 因此默认值应该总是在最后。

例子：

XML file saved at res/drawable/button.xml:

	<?xml version="1.0" encoding="utf-8"?>
	<selector xmlns:android="http://schemas.android.com/apk/res/android">
	    <item android:state_pressed="true"
	          android:drawable="@drawable/button_pressed" /> <!-- pressed -->
	    <item android:state_focused="true"
	          android:drawable="@drawable/button_focused" /> <!-- focused -->
	    <item android:state_hovered="true"
	          android:drawable="@drawable/button_focused" /> <!-- hovered -->
	    <item android:drawable="@drawable/button_normal" /> <!-- default -->
	</selector>

This layout XML applies the state list drawable to a Button:

	<Button
	    android:layout_height="wrap_content"
	    android:layout_width="wrap_content"
	    android:background="@drawable/button" />








