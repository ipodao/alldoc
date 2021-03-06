[toc]

## 设备差异

注意区别两种差异：屏幕大小和分辨率。两个屏幕可能具有相同的物理大小，但具有不同的分辨率；或具有相同的分辨率，但具有不同的物理大小。

### 术语与概念

- 屏幕大小。实际物理大小｛｛单位是英寸等长度单位｝｝，屏幕对角线的长度。为了简化，Android将屏幕大小分为四类：small, normal, large, and extra large注意：从Android 3.2 (API level 13)开始，屏幕大小分组被废弃。管理屏幕大小的新技术基于可用的屏幕宽度。参见最后一节*Declaring Tablet Layouts for Android 3.2*。
- density。屏幕上一块物理区域上像素的数量。常被称为dpi (dots per inch)。为了简化，Android将所有density分为四类：ldpi (low), mdpi (medium), hdpi (high), and xhdpi (extra high)。
- 朝向（Orientation）。This is either landscape or portrait. 朝向可能在运行时改变。
- 分辨率（Resolution）。屏幕上物理像素的总数。**应用不用关心分辨率**，只需要关心屏幕大小和density。
- Density-independent pixel (dp)。虚拟长度单位。独立于density。一个dp等价于在一个160 dpi屏幕上的物理像素，which is the baseline density assumed by the system for a "medium" density screen. 系统会根据实际density透明的处理`dp`单位的任何缩放。`dp`转换为屏幕像素很简单：`px = dp * (dpi / 160)`。例如在240 dpi屏幕上，1`dp`等于1.5倍的物理像素。使用dp能确保在不同density下显示良好。

### 支持的屏幕范围

所有泛化了的大小和分辨率都基于一个基准配置。这个基准配置是normal大小和mdpi (medium) density。选它做基准的原因是它是第一台Android设备的配置（T-Mobile G1）。

每个泛化了的大小和density都跨不同的屏幕大小和densities。Figure 1 illustrates how different sizes and densities are roughly categorized into the different size and density groups.

![](screens-ranges.png)

As you design your UI for different screen sizes, you'll discover that each design requires a minimum amount of space. 因此每个泛化的屏幕大小都有一个关联的的最小resolution。最小尺寸以"dp"为单位。

* xlarge screens are at least 960dp x 720dp
* large screens are at least 640dp x 480dp
* normal screens are at least 470dp x 320dp
* small screens are at least 426dp x 320dp

> 注意：Android 3.0之前未能很好的定义最小屏幕大小，因此你可能会遇到一些设备被错误的在normal和large之间划分。These are also based on the physical resolution of the screen, so may vary across devices—for example a 1024x720 tablet with a system bar actually has a bit less space available to the application due to it being used by the system bar.

为了优化不同屏幕大小和densities的屏幕下的显示效果，应该提供替换资源：一般说，**应该为不同的屏幕大小提供不同的布局，为不同的density提供不同的bitmap**。

不需要为每种屏幕大小和density提供一份替换资源。The system provides robust compatibility features that can handle most of the work of rendering your application on any device screen, provided that you've implemented your UI using techniques that allow it to gracefully resize （参见下面的最佳实践）。

> 注意：定义泛化大小和density的指标是相互独立的。*high-density*的屏可能只有normal大小。但*medium-density*的屏却可能有large屏。且二者可能最终的分辨率相同。the WVGA medium-density screen has a lower screen density, meaning that each pixel is physically larger and, thus, the entire screen is larger than the baseline (normal size) screen.

### Density独立

density独立，是从用户的视角看的，一个UI元素，显示在不同density的屏幕上，物理大小保持不变｛｛物理大小单位是英寸、厘米等长度单位｝｝。｛｛如果屏幕物理大小相同，仅是density不同，则UI元素在不同屏上显示效果相同，包括两方面，一方面是其自身物理大小相同，它占据屏幕的比例在各个屏幕上相同。因为这些屏幕的物理大小相同。但如果物理大小不同，虽然，UI元素自身大小在各个屏相同，但UI元素在屏幕上的比例会发生变换：在大屏上占据的屏幕比较小，在小屏上大｝｝

若不独立，一个UI元素（如按钮）在低density的屏幕上比在高density的屏幕上大（看起来大）。这些由于density导致的大小变化可能导致应用布局和可用性的变化。Figures 2 and 3 show the difference between an application when it does not provide density independence and when it does, respectively.

![](density-test-bad.png)
Figure 2. 不支持不同densities，as shown on low, medium, and high density screens.

![](density-test-good.png)
Figure 3. 支持不同densities（density独立），as shown on low, medium, and high density screens.

The Android system helps your application achieve density independence in two ways:

* 系统根据当前屏幕density缩放dp单位
* 系统根据当前屏幕density缩放drawable到实际大小

In figure 2, TextView和bitmap的大小用px指定的。In figure 3，布局用dp指定。Because the baseline for density-independent pixels is a medium-density screen, the device with a medium-density screen looks the same as it does in figure 2. For the low-density and high-density screens, however, the system scales the density-independent pixel values down and up, respectively, to fit the screen as appropriate.

多数情况下，要实现像素独立，可以使用`dp`单位，或`wrap_content`。The system then scales bitmap drawables as appropriate in order to display at the appropriate size, based on the appropriate scaling factor for the current screen's density.

However, bitmap scaling can result in blurry or pixelated bitmaps, which you might notice in the above screenshots. 为避免该问题，应该为不同density提供不同的bitmap资源。The following section describes more about how to supply alternative resources for different screen configurations.