Although the system performs scaling and resizing to make your application work on different screens, you should make the effort to optimize your application for different screen sizes and densities.

# 屏幕支持概述

## 术语与概念

* 屏幕大小  
实际物理大小｛｛单位是英寸等长度单位｝｝，屏幕对角线的长度。为了简化，Android将屏幕大小分为四类：small, normal, large, and extra large。
	> 注意：从Android 3.2 (API level 13)开始，屏幕大小分组被废弃。管理屏幕大小的新技术基于可用的屏幕宽度。参见最后一节*Declaring Tablet Layouts for Android 3.2*。
* 屏幕density  
屏幕上一块物理区域上像素的数量。常被称为dpi (dots per inch)。为了简化，Android将所有density分为四类：ldpi (low), mdpi (medium), hdpi (high), and xhdpi (extra high)。
* 朝向（Orientation）  
This is either landscape or portrait. 朝向可能在运行时改变。
* 分辨率（Resolution）  
屏幕上物理像素的总数。**应用不用关心分辨率**，只需要关心屏幕大小和density。
* Density-independent pixel (dp)  
定义UI时使用的虚拟像素单位，表达长度和位置，独立于density。一个dp等价于在一个160 dpi屏幕上的物理像素，which is the baseline density assumed by the system for a "medium" density screen. 系统会根据实际density透明的处理`dp`单位的任何缩放。 `dp`转换为屏幕像素很简单：`px = dp * (dpi / 160)`。例如在240 dpi屏幕上，1 `dp`等于1.5倍的物理像素。使用dp能确保在不同density下显示良好。

##支持的屏幕范围

The generalized sizes and densities are arranged around a baseline configuration that is a normal size and mdpi (medium) density. This baseline is based upon the screen configuration for the first Android-powered device, the T-Mobile G1, which has an HVGA screen (until Android 1.6, this was the only screen configuration that Android supported).

每个泛化了的大小和density都跨不同的屏幕大小和densities。Figure 1 illustrates how different sizes and densities are roughly categorized into the different size and density groups.

![](screens-ranges.png)

As you design your UI for different screen sizes, you'll discover that each design requires a minimum amount of space. 因此每个泛化的屏幕大小都有一个关联的的最小resolution。These minimum sizes are in "dp" units—the same units you should use when defining your layouts—which allows the system to avoid worrying about changes in screen density.

* xlarge screens are at least 960dp x 720dp
* large screens are at least 640dp x 480dp
* normal screens are at least 470dp x 320dp
* small screens are at least 426dp x 320dp

> 注意：Android 3.0之前未能很好的定义最小屏幕大小，因此你可能会遇到一些设备被错误的在normal和large之间划分。These are also based on the physical resolution of the screen, so may vary across devices—for example a 1024x720 tablet with a system bar actually has a bit less space available to the application due to it being used by the system bar.

为了优化不同屏幕大小和densities的屏幕下的显示效果，应该提供替换资源：一般说，**应该为不同的屏幕大小提供不同的布局，为不同的density提供不同的bitmap**。

不需要为每种屏幕大小和density提供一份替换资源。The system provides robust compatibility features that can handle most of the work of rendering your application on any device screen, provided that you've implemented your UI using techniques that allow it to gracefully resize （参见下面的最佳实践）.

> 注意：定义泛化大小和density的指标是相互独立的。*high-density*的屏可能只有normal大小。但*medium-density*的屏却可能有large屏。且二者可能最终的分辨率相同。the WVGA medium-density screen has a lower screen density, meaning that each pixel is physically larger and, thus, the entire screen is larger than the baseline (normal size) screen.

##Density独立

density独立，是从用户的视角看的，一个UI元素，显示在不同density的屏幕是那个，物理大小保持不变｛｛物理大小单位是英寸、厘米等长度单位｝｝。｛｛如果屏幕物理大小相同，仅是density不同，则UI元素在不同屏上显示效果相同，包括两方面，一方面是其自身物理大小相同，它占据屏幕的比例在各个屏幕上相同。因为这些屏幕的物理大小相同。但如果物理大小不同，虽然，UI元素自身大小在各个屏相同，但UI元素在屏幕上的比例会发生变换：在大屏上占据的屏幕比较小，在小屏上大｝｝

若不独立，一个UI元素（如按钮）在低density的屏幕上比在高density的屏幕上大（看起来大）。这些由于density导致的大小变化可能导致应用布局和可用性的变化。Figures 2 and 3 show the difference between an application when it does not provide density independence and when it does, respectively.

![](density-test-bad.png)
Figure 2. 不支持不同densities，as shown on low, medium, and high density screens.

![](density-test-good.png)
Figure 3. 支持不同densities（density独立），as shown on low, medium, and high density screens.

The Android system helps your application achieve density independence in two ways:

* 系统根据当前屏幕density缩放dp单位
* 系统根据当前屏幕density缩放drawable到实际大小。

In figure 2, TextView和bitmap的大小用px指定的。In figure 3，布局用dp指定。Because the baseline for density-independent pixels is a medium-density screen, the device with a medium-density screen looks the same as it does in figure 2. For the low-density and high-density screens, however, the system scales the density-independent pixel values down and up, respectively, to fit the screen as appropriate.

多数情况下，要实现像素独立，可以使用`dp`单位，或`wrap_content`。The system then scales bitmap drawables as appropriate in order to display at the appropriate size, based on the appropriate scaling factor for the current screen's density.

However, bitmap scaling can result in blurry or pixelated bitmaps, which you might notice in the above screenshots. 为避免该问题，应该为不同density提供不同的bitmap资源。The following section describes more about how to supply alternative resources for different screen configurations.

#如何支持多屏

The system handles most of the work to render your application properly on each screen configuration by scaling layouts to fit the screen size/density and scaling bitmap drawables for the screen density, as appropriate. To more gracefully handle different screen configurations, however, you should also:

* 显式声明你支持的屏幕大小  
Declaring support for different screen sizes can also affect how the system draws your application on larger screens—specifically, whether your application runs in screen compatibility mode.  
To declare the screen sizes your application supports, you should include the `<supports-screens>` element in your manifest file.
* 为不同屏幕大小定义不同布局  
Android默认会调整你的布局大小，适应当前屏幕。In most cases, this works fine. In other cases, your UI might not look as good and might need adjustments for different screen sizes. 例如在大屏下你可能需要调整元素的大小和位置。  
与大小相关的配置限定符是small, normal, large, and xlarge。例如extra large屏使用的布局`layout-xlarge/`。  
从Android 3.2 (API level 13)开始，上述大小组被弃用，你应该使用`sw<N>dp`限定符，定义布局需要的最小可用宽度。例如，如果多面板tablet布局至少需要`600dp`屏幕宽度，则布局文件应放在`layout-sw600dp/`下。
* 为不同的屏幕densities提供不同的bitmap  
Android默认会缩放你的drawables (.png, .jpg, and .gif files) 和Nine-Patch drawables (.9.png files) so that they render at the appropriate physical size on each device。例如如果你只提供了`medium screen density (mdpi)`，则在更高density屏幕下，系统会放大；在低density下系统会缩小。缩放可能代理失真。  
density相关的限定符是：ldpi (low), mdpi (medium), hdpi (high), xhdpi (extra high)。例如high-density屏的Bitmap放在`drawable-hdpi/`下。

> Note: 参见[Providing Alternative Resources](http://developer.android.com/guide/topics/resources/providing-resources.html#AlternativeResources)。

At runtime, the system ensures the best possible display on the current screen with the following procedure for any given resource:

1. 系统使用合适的替代资源  
根据当前屏的大小和density，系统选择特定大小或density的资源。
2. 若没有匹配的资源，系统使用默认的资源进行缩放。  
默认资源指没有加注配置限定符的资源。例如`drawable/`是默认的drawable资源。The system assumes that default resources are designed for the baseline screen size and density, which is a normal screen size and a medium density. As such, the system scales default density resources up for high-density screens and down for low-density screens, as appropriate.  
但如果系统未找到某个density的资源是，它不总是使用默认资源。系统可能使用其他density的资源，以期达到更好的缩放结果。例如，若要某个low-density的资源，系统会倾向于用high-density缩小，because the system can easily scale a high-density resource down to low-density by a factor of 0.5, with fewer artifacts, compared to scaling a medium-density resource by a factor of 0.75。

更多信息，参见[How Android Finds the Best-matching Resource](http://developer.android.com/guide/topics/resources/providing-resources.html#BestMatch)。

## 使用配置限定符

在`res/`创建一个新目录，格式：`<resources_name>-<qualifier>`。其中`<resources_name>`是标准资源名（如`drawable`或`layout`）。`<qualifier>`是配置限定符。  
可以使用使用多个限定符。中划线隔开。

For example, xlarge is a configuration qualifier for extra large screens. When you append this string to a resource directory name (such as layout-xlarge), it indicates to the system that these resources are to be used on devices that have an extra large screen.

限定符

* Size
	* small  
	* normal（基准大小）
	* large
	* xlarge
* Density
	* `ldpi`: Resources for low-density (ldpi) screens (~120dpi).
	* `mdpi`: Resources for medium-density (mdpi) screens (~160dpi). （基准density）
	* `hdpi`: Resources for high-density (hdpi) screens (~240dpi).
	* `xhdpi`:  Resources for extra high-density (xhdpi) screens (~320dpi).
	* `nodpi`: Resources for all densities. These are density-independent resources. The system does not scale resources tagged with this qualifier, regardless of the current screen's density.
	* `tvdpi`: 在`mdpi`和`hdpi`之间的一个分辨率。接近`213dpi`。This is not considered a "primary" density group. It is mostly intended for televisions and most apps shouldn't need it—providing mdpi and hdpi resources is sufficient for most apps and the system will scale them as appropriate. If you find it necessary to provide tvdpi resources, you should size them at a factor of 1.33*mdpi. For example, a 100px x 100px image for mdpi screens should be 133px x 133px for tvdpi.
* Orientation
	* land: Resources for screens in the landscape orientation (wide aspect ratio).
	* port: Resources for screens in the portrait orientation (tall aspect ratio).
* Aspect ratio
	* long: Resources for screens that have a significantly taller or wider aspect ratio (when in portrait or landscape orientation, respectively) than the baseline screen configuration.
	* notlong: Resources for use screens that have an aspect ratio that is similar to the baseline screen configuration.


> Note: If you're developing your application for Android 3.2 and higher, see the section about [Declaring Tablet Layouts for Android 3.2](http://developer.android.com/guide/practices/screens_support.html#DeclaringTabletLayouts) for information about new configuration qualifiers that you should use when declaring layout resources for specific screen sizes (instead of using the `size` qualifiers in table 1).

For example, the following is a list of resource directories in an application that provides different layout designs for different screen sizes and different bitmap drawables for medium, high, and extra high density screens.

	res/layout/my_layout.xml             // layout for normal screen size ("default")
	res/layout-small/my_layout.xml       // layout for small screen size
	res/layout-large/my_layout.xml       // layout for large screen size
	res/layout-xlarge/my_layout.xml      // layout for extra large screen size
	res/layout-xlarge-land/my_layout.xml // layout for extra large in landscape orientation
	
	res/drawable-mdpi/my_icon.png        // bitmap for medium density
	res/drawable-hdpi/my_icon.png        // bitmap for high density
	res/drawable-xhdpi/my_icon.png       // bitmap for extra high density

Be aware that, when the Android system picks which resources to use at runtime, it uses certain logic to determing the "best matching" resources. That is, the qualifiers you use don't have to exactly match the current screen configuration in all cases in order for the system to use them. Specifically, when selecting resources based on the size qualifiers, the system will use resources designed for a screen smaller than the current screen if there are no resources that better match (for example, a large-size screen will use normal-size screen resources if necessary). However, if the only available resources are larger than the current screen, the system will not use them and your application will crash if no other resources match the device configuration (for example, if all layout resources are tagged with the xlarge qualifier, but the device is a normal-size screen). For more information about how the system selects resources, read [How Android Finds the Best-matching Resource](http://developer.android.com/guide/topics/resources/providing-resources.html#BestMatch).

> Tip: If you have some drawable resources that the system should never scale (perhaps because you perform some adjustments to the image yourself at runtime), you should place them in a directory with the `nodpi` configuration qualifier. Resources with this qualifier are considered density-agnostic and the system will not scale them.

## 设计替换布局和drawables

### 替换布局

If your UI uses bitmaps that need to fit the size of a view even after the system scales the layout (such as the background image for a button), you should use [Nine-Patch](http://developer.android.com/guide/topics/graphics/2d-graphics.html#nine-patch) bitmap files. A Nine-Patch file is basically a PNG file in which you specific two-dimensional regions that are stretchable. When the system needs to scale the view in which the bitmap is used, the system stretches the Nine-Patch bitmap, but stretches only the specified regions. As such, you don't need to provide different drawables for different screen sizes, because the Nine-Patch bitmap can adjust to any size. You should, however, provide alternate versions of your Nine-Patch files for different screen densities.

### 替换Drawable

Almost every application should have alternative drawable resources for different screen densities, because almost every application has a launcher icon and that icon should look good on all screen densities. Likewise, if you include other bitmap drawables in your application (such as for menu icons or other graphics in your application), you should provide alternative versions or each one, for different densities.

> Note: You only need to provide density-specific drawables for bitmap files (.png, .jpg, or .gif) and Nine-Path files (.9.png). If you use XML files to define shapes, colors, or other drawable resources, you should put one copy in the default drawable directory (drawable/).

要为不同density创建不同的bitmap drawables，四种泛化的density之间的比例是3:4:6:8。For example, if you have a bitmap drawable that's 48x48 pixels for medium-density screen (the size for a launcher icon), all the different sizes should be:

* 36x36 for low-density
* 48x48 for medium-density
* 72x72 for high-density
* 96x96 for extra high-density

![](screens-densities.png)  
Figure 4. Relative sizes for bitmap drawables that support each density.

For more information about designing icons, see the [Icon Design Guidelines](http://developer.android.com/guide/practices/ui_guidelines/icon_design.html), which includes size information for various bitmap drawables, such as launcher icons, menu icons, status bar icons, tab icons, and more.

# Declaring Tablet Layouts for Android 3.2

For the first generation of tablets running Android 3.0, the proper way to declare tablet layouts was to put them in a directory with the xlarge configuration qualifier (for example, res/layout-xlarge/). In order to accommodate other types of tablets and screen sizes—in particular, 7" tablets—Android 3.2 introduces a new way to specify resources for more discrete screen sizes. 新技术基于你的布局所需的控件大小（如宽度600dp）。rather than trying to make your layout fit the generalized size groups (such as large or xlarge).

The reason designing for 7" tablets is tricky when using the generalized size groups is that a 7" tablet is technically in the same group as a 5" handset (the large group). While these two devices are seemingly close to each other in size, the amount of space for an application's UI is significantly different, as is the style of user interaction. 因此7寸和6寸的屏不应该使用相同的布局。To make it possible for you to provide different layouts for these two kinds of screens, Android now allows you to specify your layout resources based on the width and/or height that's actually available for your application's layout, 单位`dp`。

For example, after you've designed the layout you want to use for tablet-style devices, you might determine that the layout stops working well when the screen is less than 600dp wide. This threshold thus becomes the minimum size that you require for your tablet layout. As such, you can now specify that these layout resources should be used only when there is at least 600dp of width available for your application's UI.

两种选择：选择一个宽度作为最小宽度，然后设计布局。或者，完成布局后测试能兼容的最小宽度。

> Note: Remember that all the figures used with these new size APIs are density-indpendent pixel (dp) values and your layout dimensions should also always be defined using dp units, because what you care about is the amount of screen space available after the system accounts for screen density (as opposed to using raw pixel resolution).

## 使用新的大小限定符

> Note: 你指定的限制不是屏幕大小，而是可用大小。部分系统可能用于系统UI（下面的是system bar和上面的status bar）。这部分大小你不必考虑。Action Bar算作你应用的一部分，要考虑。

Table 2. New configuration qualifers for screen size (introduced in Android 3.2).

* 最小宽度设备smallestWidth：`sw<N>dp`
例如：`sw600dp`、`sw720dp`  
The fundamental size of a screen, as indicated by the **shortest** dimension of the available screen area. 设备`smallestWidth`是屏幕可用的高和宽的最小值。利用该值，可以保证，不管使用何种朝向，应用至少可以使用`<N> dps`。改变朝向不改变`smallestWidth`的值。  
For example, if your layout requires that its smallest dimension of screen area be at least 600 dp at all times, then you can use this qualifer to create the layout resources, res/layout-sw600dp/. The system will use these resources only when the smallest dimension of available screen is at least 600dp, regardless of whether the 600dp side is the user-perceived height or width.   
 Using smallestWidth to determine the general screen size is useful because width is often the driving factor in designing a layout. A UI will often scroll vertically, but have fairly hard constraints on the minimum space it needs horizontally. The available width is also the key factor in determining whether to use a one-pane layout for handsets or multi-pane layout for tablets. Thus, you likely care most about what the smallest possible width will be on each device.
* 可用屏幕宽度：`w<N>dp`  
例子：`w720dp`、`w1024dp`。  
Specifies a minimum available width in dp units at which the resources should be used—defined by the <N> value. The system's corresponding value for the width changes when the screen's orientation switches between landscape and portrait to reflect the current actual width that's available for your UI.  
This is often useful to determine whether to use a multi-pane layout, because even on a tablet device, you often won't want the same multi-pane layout for portrait orientation as you do for landscape. Thus, you can use this to specify the minimum width required for the layout, instead of using both the screen size and orientation qualifiers together.